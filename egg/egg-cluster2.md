# 浅析egg-cluster多进程管理模块（二）

## egg-cluster多进程管理模块

### 大致工作流程

1. 执行detectPort函数，侦测0端口是否空闲，若空闲，执行master.forkAgentWorker方法

    // egg-cluster@1.9.0 lib/master.js
    class Master extends EventEmitter {
      constructor(options) {

        // ...

        detectPort((err, port) => {
          /* istanbul ignore if */
          if (err) {
            err.name = 'ClusterPortConflictError';
            err.message = '[master] try get free port error, ' + err.message;
            this.logger.error(err);
            process.exit(1);
            return;
          }
          this.options.clusterPort = port;
          this.forkAgentWorker();
        });

      }

      // ...

    }

2. forkAgentWorker方法执行过程中，将启动agent进程，随后促使master多进程管理类发送"agent-start"消息

    // egg-cluster@1.9.0 lib/master.js
    class Master extends EventEmitter {

      // ...

      forkAgentWorker() {
        this.agentStartTime = Date.now();

        const args = [ JSON.stringify(this.options) ];
        // worker name     | debug port
        // ---             | ---
        // agent_worker#0  | 5856
        // master          | 5857
        // app_worker#0    | 5858
        // app_worker#1    | 5859
        // ...
        const opt = { execArgv: process.execArgv.concat([ '--debug-port=5856' ]) };
        const agentWorker = this.agentWorker = childprocess.fork(agentWorkerFile, args, opt);
        agentWorker.id = ++this.agentWorkerIndex;
        this.log('[master] agent_worker#%s:%s start with clusterPort:%s',
          agentWorker.id, agentWorker.pid, this.options.clusterPort);

        // forwarding agent' message to messenger
        // 将agent进程获得的消息转发给worker进程，以使worker进程发送同样的消息，供外部订阅
        agentWorker.on('message', msg => {
          if (typeof msg === 'string') msg = { action: msg, data: msg };
          msg.from = 'agent';
          this.messenger.send(msg);
        });
        agentWorker.on('error', err => {
          err.name = 'AgentWorkerError';
          err.id = agentWorker.id;
          err.pid = agentWorker.pid;
          this.logger.error(err);
        });
        // agent exit message
        // agentWorker退出时，促使master多进程管理实例发送"agent-exit"消息，以执行onAgentExit钩子函数
        // agentWorker退出时，人工处理其错误（参考egg文档）；worker进程异常退出时，重新创建一个worker进程
        agentWorker.once('exit', (code, signal) => {
          this.messenger.send({
            action: 'agent-exit',
            data: { code, signal },
            to: 'master',
            from: 'agent',
          });
        });
      }

      // ...

    }

    // egg-cluster@1.9.0 lib/agent_worker.js
    agent.ready(() => {
      agent.removeListener('error', startErrorHandler);
      process.send({ action: 'agent-start', to: 'master' });
    });

3. 由master多进程管理类订阅"agent-start"消息，执行forkAppWorkers方法，启动worker进程，并创建http或https服务

    // egg-cluster@1.9.0 lib/master.js
    class Master extends EventEmitter {
      constructor(options) {

        // ...

        this.on('agent-start', this.onAgentStart.bind(this));

        this.once('agent-start', this.forkAppWorkers.bind(this));

      }

      // ...

      forkAppWorkers() {
        this.appStartTime = Date.now();
        this.isAllAppWorkerStarted = false;
        this.startSuccessCount = 0;

        this.workers = new Map();

        const args = [ JSON.stringify(this.options) ];
        this.log('[master] start appWorker with args %j', args);
        cfork({
          exec: appWorkerFile,
          args,
          silent: false,
          count: this.options.workers,
          // don't refork in local env
          refork: this.isProduction,
        });

        cluster.on('fork', worker => {
          this.workers.set(worker.process.pid, worker);
          worker.on('message', msg => {
            if (typeof msg === 'string') msg = { action: msg, data: msg };
            msg.from = 'app';
            this.messenger.send(msg);
          });
          this.log('[master] app_worker#%s:%s start, state: %s, current workers: %j',
            worker.id, worker.process.pid, worker.state, Object.keys(cluster.workers));
        });
        cluster.on('disconnect', worker => {
          this.logger.info('[master] app_worker#%s:%s disconnect, suicide: %s, state: %s, current workers: %j',
            worker.id, worker.process.pid, worker.exitedAfterDisconnect, worker.state, Object.keys(cluster.workers));
        });
        cluster.on('exit', (worker, code, signal) => {
          this.messenger.send({
            action: 'app-exit',
            data: { workerPid: worker.process.pid, code, signal },
            to: 'master',
            from: 'app',
          });
        });
        // worker进程监听端口，促使master多进程管理类发送"app-start"消息，执行onAppStart函数
        cluster.on('listening', (worker, address) => {
          this.messenger.send({
            action: 'app-start',
            data: { workerPid: worker.process.pid, address },
            to: 'master',
            from: 'app',
          });
        });
      }

    }

    // egg-cluster@1.9.0 lib/app_worker.js
    function startServer() {
      app.removeListener('error', startErrorHandler);
      app.removeListener('startTimeout', startTimeoutHandler);

      let server;
      if (options.https) {
        server = require('https').createServer({
          key: fs.readFileSync(options.key),
          cert: fs.readFileSync(options.cert),
        }, app.callback());
      } else {
        server = require('http').createServer(app.callback());
      }

      // emit `server` event in app
      app.emit('server', server);

      if (options.sticky) {
        server.listen(0, '127.0.0.1');
        // Listen to messages sent from the master. Ignore everything else.
        process.on('message', (message, connection) => {
          if (message !== 'sticky-session:connection') {
            return;
          }

          // Emulate a connection event on the server by emitting the
          // event with the connection the master sent us.
          server.emit('connection', connection);
          connection.resume();
        });
      } else {
        if (listenConfig.path) {
          server.listen(listenConfig.path);
        } else {
          if (typeof port !== 'number') {
            consoleLogger.error('[app_worker] port should be number, but got %s', port);
            process.exit(1);
          }
          const args = [ port ];
          if (listenConfig.hostname) args.push(listenConfig.hostname);
          debug('listen options %s', args);
          server.listen(...args);
        }
      }
    }

4. 由master多进程管理类订阅"agent-start"消息，执行onAgentStart钩子方法
  - 由agent进程促使worker进程发送"egg-pids"、"agent-start"消息，供外部订阅

    // egg-cluster@1.9.0 lib/master.js
    class Master extends EventEmitter {
      // ...

      onAgentStart() {
        this.messenger.send({ action: 'egg-pids', to: 'app', data: [ this.agentWorker.pid ] });
        this.messenger.send({ action: 'agent-start', to: 'app' });
        this.logger.info('[master] agent_worker#%s:%s started (%sms)',
          this.agentWorker.id, this.agentWorker.pid, Date.now() - this.agentStartTime);
      }
    }

5. worker进程启动的http或https服务监听端口时，由worker进程促使master发送"app-start"消息

    // egg-cluster@1.9.0 lib/master.js
    class Master extends EventEmitter {

      // ...

      forkAppWorkers() {
        this.appStartTime = Date.now();
        this.isAllAppWorkerStarted = false;
        this.startSuccessCount = 0;

        this.workers = new Map();

        const args = [ JSON.stringify(this.options) ];
        this.log('[master] start appWorker with args %j', args);
        cfork({
          exec: appWorkerFile,
          args,
          silent: false,
          count: this.options.workers,
          // don't refork in local env
          refork: this.isProduction,
        });

        cluster.on('fork', worker => {
          this.workers.set(worker.process.pid, worker);
          worker.on('message', msg => {
            if (typeof msg === 'string') msg = { action: msg, data: msg };
            msg.from = 'app';
            this.messenger.send(msg);
          });
          this.log('[master] app_worker#%s:%s start, state: %s, current workers: %j',
            worker.id, worker.process.pid, worker.state, Object.keys(cluster.workers));
        });
      }

    }

6. 由master多进程管理类订阅"appstart"消息，执行onAppStart方法
  - 由worker进程促使agent进程发送"egg-pids"消息，供外部订阅
  - 所需进程均已启动，且在sticky模式下，执行startMasterSocketServer方法
    - master多进程管理类中再行启动tcp服务监听实际端口，由特定worker进程发送消息订阅该socket

    // egg-cluster@1.9.0 lib/master.js
    class Master extends EventEmitter {
      constructor(options) {

        // ...

        this.on('app-start', this.onAppStart.bind(this));

      }

      onAppStart(data) {
        const worker = this.workers.get(data.workerPid);
        const address = data.address;

        // ignore unspecified port
        // and it is ramdom port when use sticky
        if (!this.options.sticky
          && !isUnixSock(address)
          && (String(address.port) !== String(this[REALPORT]))) {
          return;
        }

        // send message to agent with alive workers
        this.messenger.send({
          action: 'egg-pids',
          to: 'agent',
          data: getListeningWorker(this.workers),
        });

        this.startSuccessCount++;

        const remain = this.isAllAppWorkerStarted ? 0 : this.options.workers - this.startSuccessCount;
        this.log('[master] app_worker#%s:%s started at %s, remain %s (%sms)',
          worker.id, data.workerPid, address.port, remain, Date.now() - this.appStartTime);

        if (this.isAllAppWorkerStarted || this.startSuccessCount < this.options.workers) {
          return;
        }

        this.isAllAppWorkerStarted = true;

        address.protocal = this.options.https ? 'https' : 'http';
        address.port = this.options.sticky ? this[REALPORT] : address.port;
        this[APP_ADDRESS] = getAddress(address);

        if (this.options.sticky) {
          this.startMasterSocketServer(err => {
            if (err) return this.ready(err);
            this.ready(true);
          });
        } else {
          this.ready(true);
        }
      }

      startMasterSocketServer(cb) {
        /* from Node documents
        If pauseOnConnect is set to true, then the socket associated with each incoming connection 
        will be paused, and no data will be read from its handle. This allows connections to be passed 
        between processes without any data being read by the original process. To begin reading data 
        from a paused socket, call socket.resume().
        */

        // Create the outside facing server listening on our port.
        require('net').createServer({ pauseOnConnect: true }, connection => {
          // We received a connection and need to pass it to the appropriate
          // worker. Get the worker for this connection's source IP and pass
          // it the connection.
          const worker = this.stickyWorker(connection.remoteAddress);
          worker.send('sticky-session:connection', connection);
        }).listen(this[REALPORT], cb);
      }

      // 由IP地址启动同一个worker进程处理MasterSocketServer的句柄
      stickyWorker(ip) {
        const workerNumbers = this.options.workers;
        const ws = Array.from(this.workers.keys());

        let s = '';
        for (let i = 0; i < ip.length; i++) {
          if (!isNaN(ip[i])) {
            s += ip[i];
          }
        }
        s = Number(s);
        const pid = ws[s % workerNumbers];
        return this.workers.get(pid);
      }
    }

7. 执行this.ready(cb)添加的回调cb
    - 主进程、agent进程、worker进程发送"egg-ready"消息，供外部订阅

    // egg-cluster@1.9.0 lib/master.js
    class Master extends EventEmitter {
      constructor(options) {

        // ...

        this.ready(() => {
          this.isStarted = true;
          const stickyMsg = this.options.sticky ? ' with STICKY MODE!' : '';
          this.logger.info('[master] %s started on %s (%sms)%s',
            frameworkPkg.name, this[APP_ADDRESS], Date.now() - startTime, stickyMsg);

          // 主进程、agent进程、worker进程发送"egg-ready"消息，由egg-cluster模块使用者订阅
          const action = 'egg-ready';
          this.messenger.send({ action, to: 'parent' });
          this.messenger.send({ action, to: 'app', data: this.options });
          this.messenger.send({ action, to: 'agent', data: this.options });
        });

      }

      onAppStart(data) {

        // ...

        if (this.options.sticky) {
          this.startMasterSocketServer(err => {
            if (err) return this.ready(err);
            this.ready(true);
          });
        } else {
          this.ready(true);
        }
      }

### 消息模型

egg-cluster模块设计了messager类用于转发消息，促使订阅该消息的进程执行相应的逻辑。

消息对象格式如同{ from:"agent", to:"app" }，其意义是促使worker进程发送该消息，同时当worker订阅该消息将执行其设定的监听函数。

通过utils/messager.js脚本可促使master主进程(即parent)、worker进程或agent进程执行监听函数，或者促使多进程管理类master执行相应事件的绑定函数（在egg-cluster模块内部应用是实现钩子，以特定时机执行某函数）。多进程管理类搭建在EventEmitter事件模型的基础上，parent进程、agent进程、worker进程则通过IPC通道实现发送消息及执行回调。

    const cluster = require('cluster');
    const sendmessage = require('sendmessage');// 促使子进程通过IPC发送消息或触发事件，两者均执行监听函数
    const debug = require('debug')('egg-cluster:messenger');

    class Messenger {

      constructor(master) {
        this.master = master;
        process.on('message', msg => {
          msg.from = 'parent';
          this.send(msg);
        });
      }

      /**
      * send message
      * @param {Object} data message body
      *  - {String} from from who
      *  - {String} to to who
      */
      send(data) {
        if (!data.from) {
          data.from = 'master';
        }

        // default from -> to rules
        if (!data.to) {
          if (data.from === 'agent') data.to = 'app';
          if (data.from === 'app') data.to = 'agent';
          if (data.from === 'parent') data.to = 'master';
        }

        // app -> master
        // agent -> master
        if (data.to === 'master') {
          debug('%s -> master, data: %j', data.from, data);
          // app/agent to master
          this.sendToMaster(data);
          return;
        }

        // master -> parent
        // app -> parent
        // agent -> parent
        if (data.to === 'parent') {
          debug('%s -> parent, data: %j', data.from, data);
          this.sendToParent(data);
          return;
        }

        // parent -> master -> app
        // agent -> master -> app
        if (data.to === 'app') {
          debug('%s -> %s, data: %j', data.from, data.to, data);
          this.sendToAppWorker(data);
          return;
        }

        // parent -> master -> agent
        // app -> master -> agent，可能不指定 to
        if (data.to === 'agent') {
          debug('%s -> %s, data: %j', data.from, data.to, data);
          this.sendToAgentWorker(data);
          return;
        }
      }

      /**
      * send message to master self
      * @param {Object} data message body
      */
      sendToMaster(data) {
        this.master.emit(data.action, data.data);
      }

      /**
      * send message to parent process
      * @param {Object} data message body
      */
      sendToParent(data) {
        process.send && process.send(data);
      }

      /**
      * send message to app worker
      * @param {Object} data message body
      */
      sendToAppWorker(data) {
        for (const id in cluster.workers) {
          const worker = cluster.workers[id];
          if (worker.state === 'disconnected') {
            continue;
          }
          // check receiverPid
          if (data.receiverPid && data.receiverPid !== String(worker.process.pid)) {
            continue;
          }
          sendmessage(worker, data);
        }
      }

      /**
      * send message to agent worker
      * @param {Object} data message body
      */
      sendToAgentWorker(data) {
        if (this.master.agentWorker) {
          sendmessage(this.master.agentWorker, data);
        }
      }

    }

    module.exports = Messenger;

### 疑问

1. detectPort侦测的是0端口，startMasterSocketServer方法又启动net服务监听实际端口，egg框架通过代理转发了服务？使用特定进程监听实际端口用于获取session？

2. require("egg-cluster")返回值为{agentWorker,workers,messager}，即agent进程、以进程pid为键的worker进程集合、及事件模型，egg框架在实际使用过程中，怎样对开发者暴露agent用于操控agent进程？同时将messager挂载到agent对象上？

3. 消息模型在开发中的借鉴意义？

4. 进程间的通信工作为什么不能借助cluster模块本身完成？工作进程断线后、重启一个工作进程是否由cluster模块处理？

5. 与"midway-cluster"模块的比较？
