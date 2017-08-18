# 浅析egg-cluster多进程管理模块（一）

## nodejs/cluster集群模块

cluster模块采用Master-Worker模式，以主进程操控子进程的方式启动多个http或https服务器。主要致力于解决两个问题：

  - 多个工作进程的http或https服务监听同一个端口；

  - http或https服务器与cluster内部tcp服务间的通信。

大致实现思路是：
  - 在工作进程http或https服务的listen方法执行时，启动主进程内部的tcp服务，置空工作进程http或https服务的listen方法；

  - 主进程tcp服务"connection"事件时，促使工作进程将主进程服务发出的net#Server交由工作进程的http或https服务处理。

### 多进程启动服务

按DavidCai1993在《[通过源码解析 Node.js 中 cluster 模块的主要功能实现](https://cnodejs.org/topic/56e84480833b7c8a0492e20c)》这篇文章中的表述，当主进程调用cluster.fork方法执行子进程脚本时，node在net模块的实现中筛分了net#Server实例listen方法的执行场景，以策略模式对不同场景予以不同处理。

cluster.fork执行场景下，当主进程的tcp服务尚未创建时，则予以创建，用以实际处理监听端口的职能；同时worker进程下启动的http/https服务，其监听端口行为也变得无效。其余场景仍采用net#Server实例listen方法原有的执行逻辑。

node@8.3.0中的表现是，根据net#Server实例listen方法的参数筛分场景，后续执行逻辑相同。个中情形又不同于DavidCai1993的说法。

node@8.3.0版本中，通过环境变量NODE_UNIQUE_ID决定cluster模块实际调用哪份脚本文件。NODE_UNIQUE_ID初始值为0，require("cluster")模块将调用lib/internal/cluster/master.js脚本，cluster.fork将使NODE_UNIQUE_ID自增1，并创建工作进程；随后在其他脚本文件中require("cluster")模块，因NODE_UNIQUE_ID为真值，将调用lib/internal/cluster/child.js脚本。如使用者require("cluster")模块fork工作进程时，调用的是master脚本，node内置的net模块require("cluster")时，调用的是child脚本。

    const childOrMaster = 'NODE_UNIQUE_ID' in process.env ? 'child' : 'master';
    module.exports = require(`internal/cluster/${childOrMaster}`);

同时，master脚本的isMaster属性为真值，cluster._getServer方法也不存在；child脚本的isMaster属性为否值，存在cluster._getServer方法。对于cluster.fork工作进程中创建的net#Server实例，在其执行listen方法时，将调用cluster._getServer方法。

    // node@8.3.0 lib/net.js
    function listenInCluster(server, address, port, addressType, backlog, fd, exclusive) {
      exclusive = !!exclusive;

      if (!cluster) cluster = require('cluster');

      if (cluster.isMaster || exclusive) {
        // Will create a new handle
        // _listen2 sets up the listened handle, it is still named like this
        // to avoid breaking code that wraps this method
        server._listen2(address, port, addressType, backlog, fd);
        return;
      }

      const serverQuery = {
        address: address,
        port: port,
        addressType: addressType,
        fd: fd,
        flags: 0
      };

      // Get the master's server handle, and listen on it
      cluster._getServer(server, serverQuery, listenOnMasterHandle);

      function listenOnMasterHandle(err, handle) {
        // EADDRINUSE may not be reported until we call listen(). To complicate
        // matters, a failed bind() followed by listen() will implicitly bind to
        // a random port. Ergo, check that the socket is bound to the expected
        // port before calling listen().
        //
        // FIXME(bnoordhuis) Doesn't work for pipe handles, they don't have a
        // getsockname() method. Non-issue for now, the cluster module doesn't
        // really support pipes anyway.
        if (err === 0 && port > 0 && handle.getsockname) {
          var out = {};
          err = handle.getsockname(out);
          if (err === 0 && port !== out.port)
            err = uv.UV_EADDRINUSE;
        }

        if (err) {
          var ex = exceptionWithHostPort(err, 'bind', address, port);
          return server.emit('error', ex);
        }

        // Reuse master's server handle
        server._handle = handle;
        // _listen2 sets up the listened handle, it is still named like this
        // to avoid breaking code that wraps this method
        server._listen2(address, port, addressType, backlog, fd);
      }
    }

lib/internal/cluster/child.js脚本中，cluster._getServer方法将促使工作进程send消息，该消息通过lib/internal/utils.js模块转化为"internalMessage"消息。内部消息的判断凭据是子对象发送的message消息对象含有cmd属性，且该属性以"NODE_"前缀起始。

    // node@8.3.0 lib/internal/cluster/child.js
    // cb回调为listenInCluster函数中的listenOnMasterHandle函数
    cluster._getServer = function(obj, options, cb) {
      const indexesKey = [options.address,
                          options.port,
                          options.addressType,
                          options.fd ].join(':');

      // ...

      const message = util._extend({
        act: 'queryServer',
        index: indexes[indexesKey],
        data: null
      }, options);

      // ...

      // 工作进程发送消息，次参作为回调添加到lib/internal/utils.js模块callbacks回调队列中
      send(message, (reply, handle) => {
        if (typeof obj._setServerData === 'function')
          obj._setServerData(reply.data);

        if (handle)
          shared(reply, handle, indexesKey, cb);  // Shared listen socket.
        else
          rr(reply, indexesKey, cb);              // Round-robin.
      });

      // ...
    };

    // ...

    function send(message, cb) {
      return sendHelper(process, message, null, cb);
    }

    // node@8.3.0 lib/internal/cluster/utils.js
    // 将工作进程发送的消息转化为内部消息
    // 将cb添加到callbacks中
    var seq = 0;
    function sendHelper(proc, message, handle, cb) {
      if (!proc.connected)
        return false;

      // Mark message as internal. See INTERNAL_PREFIX in lib/child_process.js
      message = util._extend({ cmd: 'NODE_CLUSTER' }, message);

      if (typeof cb === 'function')
        callbacks[seq] = cb;

      message.seq = seq;
      seq += 1;
      return proc.send(message, handle);
    }

而在lib/internal/cluster/master.js脚本中，cluster.fork方法执行时，将促成新创建的工作进程订阅内部消息，借由通过onmessage函数处理该内部消息。又因lib/internal/cluster/child.js模块发送的内部消息，其act属性为"queryServer"，所以在onmessage函数处理过程中，最终将调用queryServer函数。

    // node@8.3.0 lib/internal/cluster/master.js
    cluster.fork = function(env) {
      cluster.setupMaster();
      const id = ++ids;
      const workerProcess = createWorkerProcess(id, env);
      const worker = new Worker({
        id: id,
        process: workerProcess
      });

      // ...

      worker.process.on('internalMessage', internal(worker, onmessage));
      // ...
    };

    function onmessage(message, handle) {
      const worker = this;

      // ...

      else if (message.act === 'queryServer')
        queryServer(worker, message);
      
      // ...
    }

    // node@8.3.0 lib/internal/cluster/utils.js
    // 以cb回调即onmessage函数处理消息和句柄
    function internal(worker, cb) {
      return function onInternalMessage(message, handle) {
        if (message.cmd !== 'NODE_CLUSTER')
          return;

        var fn = cb;

        // ...

        fn.apply(worker, arguments);
      };
    }

关于queryServer函数，它的工作内容即是在首次fork工作进程、且在该工作进程中调用net#Server实例的listen方法时，创建cluster模块内部的tcp服务。因工作进程发送的内部消息中，其启动服务的配置项address、port、addressType、fd均由用户配置，所以内部Tcp服务只会创建一个。该tcp服务在lib/internal/cluster/round_robin_handle.js模块中通过实例化RoundRobinHandle
改造函数时创建。随后再次fork工作进程时，queryServer函数将重用缓存的RoundRobinHandle
实例，而不是创建新的tcp服务。
又因内部tcp服务在master进程中创建，而master进程的'NODE_UNIQUE_ID'环境变量为否值，该net#Server实例执行listen方法时，将以创建Tcp实例的方式赋值net#Server实例的_handle属性，再通过该Tcp实例完成端口监听。

    // node@8.3.0 lib/internal/cluster/master.js
    function queryServer(worker, message) {
      // Stop processing if worker already disconnecting
      if (worker.exitedAfterDisconnect)
        return;

      const args = [message.address,
                    message.port,
                    message.addressType,
                    message.fd,
                    message.index];
      const key = args.join(':');
      var handle = handles[key];

      if (handle === undefined) {
        var constructor = RoundRobinHandle;
        // UDP is exempt from round-robin connection balancing for what should
        // be obvious reasons: it's connectionless. There is nothing to send to
        // the workers except raw datagrams and that's pointless.
        if (schedulingPolicy !== SCHED_RR ||
            message.addressType === 'udp4' ||
            message.addressType === 'udp6') {
          constructor = SharedHandle;
        }

        handles[key] = handle = new constructor(key,
                                                message.address,
                                                message.port,
                                                message.addressType,
                                                message.fd,
                                                message.flags);
      }

      // ...

    }

    // node@8.3.0 lib/internal/cluster/round_robin_handle.js
    function RoundRobinHandle(key, address, port, addressType, fd) {
      this.key = key;
      this.all = {};
      this.free = [];
      this.handles = [];
      this.handle = null;
      this.server = net.createServer(assert.fail);

      if (fd >= 0)
        this.server.listen({ fd });
      else if (port >= 0)
        this.server.listen(port, address);
      else
        this.server.listen(address);  // UNIX socket path.

      // ...

    }

    // node@8.3.0 lib/net.js
    // 主进程启动内部tcp服务时，fd为undefined
    function listenInCluster(server, address, port, addressType, backlog, fd, exclusive) {
      exclusive = !!exclusive;

      if (!cluster) cluster = require('cluster');

      // cluster内部tcp服务由master进程启动
      if (cluster.isMaster || exclusive) {
        // Will create a new handle
        // _listen2 sets up the listened handle, it is still named like this
        // to avoid breaking code that wraps this method
        server._listen2(address, port, addressType, backlog, fd);
        return;
      }

      // ...

    }

    Server.prototype._listen2 = setupListenHandle;  // legacy alias

    // 启动tcp服务
    function setupListenHandle(address, port, addressType, backlog, fd) {
      debug('setupListenHandle', address, port, addressType, backlog, fd);

      // If there is not yet a handle, we need to create one and bind.
      // In the case of a server sent via IPC, we don't need to do this.
      if (this._handle) {
        debug('setupListenHandle: have a handle already');
      } else {
        debug('setupListenHandle: create a handle');

        // ...

        if (rval === null)
          rval = createServerHandle(address, port, addressType, fd);

        if (typeof rval === 'number') {
          var error = exceptionWithHostPort(rval, 'listen', address, port);
          process.nextTick(emitErrorNT, this, error);
          return;
        }
        this._handle = rval;
      }

      this[async_id_symbol] = getNewAsyncId(this._handle);
      this._handle.onconnection = onconnection;
      this._handle.owner = this;

      // Use a backlog of 512 entries. We pass 511 to the listen() call because
      // the kernel does: backlogsize = roundup_pow_of_two(backlogsize + 1);
      // which will thus give us a backlog of 512 entries.
      var err = this._handle.listen(backlog || 511);

      // ...
    }

    // 创建Tcp实例
    function createServerHandle(address, port, addressType, fd) {
      var err = 0;
      // assign handle in listen, and clean up if bind or listen fails
      var handle;

      var isTCP = false;
      if (typeof fd === 'number' && fd >= 0) {
        // ...
      } else if (port === -1 && addressType === -1) {
        // ...
      } else {
        handle = new TCP();
        isTCP = true;
      }

      // ...

      return handle;
    }

每次cluster.fork工作进程，并调用net#Server实例的listen方法时，都会执行RoundRobinHandle.add方法，促使工作进程发送{ack:message.seq}内部消息。基于前述lib/internal/cluster/master.js同样一套侦听内部消息的逻辑，工作进程在侦听{ack}消息过程，将调用lib/internal/cluster/utils.js模块中存储的回调函数。其中，消息对象的ack属性，用于指定将调用哪个回调函数。该回调函数即cluster._getServer方法调用send方法时传入的次参，从而间接调用rr函数。

    // node@8.3.0 lib/internal/cluster/master.js
    function queryServer(worker, message) {
      
      // ...

      // Set custom server data
      handle.add(worker, (errno, reply, handle) => {
        reply = util._extend({
          errno: errno,
          key: key,
          ack: message.seq,
          data: handles[key].data
        }, reply);

        if (errno)
          delete handles[key];  // Gives other workers a chance to retry.

        send(worker, reply, handle);
      });
    }

    // node@8.3.0 lib/internal/cluster/round_robin_handle.js
    RoundRobinHandle.prototype.add = function(worker, send) {
      assert(worker.id in this.all === false);
      this.all[worker.id] = worker;

      const done = () => {
        if (this.handle.getsockname) {
          const out = {};
          this.handle.getsockname(out);
          // TODO(bnoordhuis) Check err.
          send(null, { sockname: out }, null);
        } else {
          send(null, null, null);  // UNIX socket.
        }

        this.handoff(worker);  // In case there are connections pending.
      };

      if (this.server === null)
        return done();

      // Still busy binding.
      this.server.once('listening', done);
      // ...
    };

    // node@8.3.0 lib/internal/cluster/utils.js
    // 工作进程发送{ack}消息时，将触发执行utils模块中存储的回调函数
    function internal(worker, cb) {
      return function onInternalMessage(message, handle) {
        // ...

        if (message.ack !== undefined && callbacks[message.ack] !== undefined) {
          fn = callbacks[message.ack];
          delete callbacks[message.ack];
        }

        fn.apply(worker, arguments);
      };
    }

    // node@8.3.0 lib/internal/cluster/child.js
    cluster._getServer = function(obj, options, cb) {

      // ...

      send(message, (reply, handle) => {
        if (typeof obj._setServerData === 'function')
          obj._setServerData(reply.data);

        if (handle)
          shared(reply, handle, indexesKey, cb);  // Shared listen socket.
        else
          rr(reply, indexesKey, cb);              // Round-robin.
      });

      // ...
    };

rr函数执行过程中，将构建handle={listen,close}对象，并作为次参传入lib/net.js模块中listenOnMasterHandle函数里，并调用listenOnMasterHandle函数。随着listenOnMasterHandle函数的执行，工作进程创建的net#Server实例将添加_handle属性，等到程序调用net#Server实例的_listen2方法时，因为存在_handle属性，该方法内不执行任何有意义的脚本，即不触发net#Server实例原listen方法端口监听动作。

    // node@8.3.0 lib/internal/cluster/child.js
    // 参数cb为lib/net.js模块中的listenOnMasterHandle函数
    function rr(message, indexesKey, cb) {
      if (message.errno)
        return cb(message.errno, null);

      var key = message.key;

      function listen(backlog) {
        // TODO(bnoordhuis) Send a message to the master that tells it to
        // update the backlog size. The actual backlog should probably be
        // the largest requested size by any worker.
        return 0;
      }

      function close() {
        // lib/net.js treats server._handle.close() as effectively synchronous.
        // That means there is a time window between the call to close() and
        // the ack by the master process in which we can still receive handles.
        // onconnection() below handles that by sending those handles back to
        // the master.
        if (key === undefined)
          return;

        send({ act: 'close', key });
        delete handles[key];
        delete indexes[indexesKey];
        key = undefined;
      }

      function getsockname(out) {
        if (key)
          util._extend(out, message.sockname);

        return 0;
      }

      // Faux handle. Mimics a TCPWrap with just enough fidelity to get away
      // with it. Fools net.Server into thinking that it's backed by a real
      // handle. Use a noop function for ref() and unref() because the control
      // channel is going to keep the worker alive anyway.
      const handle = { close, listen, ref: noop, unref: noop };

      if (message.sockname) {
        handle.getsockname = getsockname;  // TCP handles only.
      }

      assert(handles[key] === undefined);
      handles[key] = handle;
      cb(0, handle);
    }

    // node@8.3.0 lib/net.js
    function listenOnMasterHandle(err, handle) {
      
      // ...

      // Reuse master's server handle
      server._handle = handle;
      // _listen2 sets up the listened handle, it is still named like this
      // to avoid breaking code that wraps this method
      server._listen2(address, port, addressType, backlog, fd);
    }

    Server.prototype._listen2 = setupListenHandle;  // legacy alias

    function setupListenHandle(address, port, addressType, backlog, fd) {
      debug('setupListenHandle', address, port, addressType, backlog, fd);

      // If there is not yet a handle, we need to create one and bind.
      // In the case of a server sent via IPC, we don't need to do this.
      if (this._handle) {
        debug('setupListenHandle: have a handle already');
      }

      // ...

    }

随后工作进程启动的http或https服务将侦测到"listening"事件，触发工作进程发送{act:"listening"}内部消息，基于前述lib/internal/cluster/master.js同样一套侦听内部消息的逻辑，通过onmessage函数将驱动执行listening函数，触发"linstening"事件，以使开发者订阅"listening"消息。

    // node@8.3.0 lib/internal/cluster/child.js
    cluster._getServer = function(obj, options, cb) {

      // ...

      obj.once('listening', () => {
        cluster.worker.state = 'listening';
        const address = obj.address();
        message.act = 'listening';
        message.port = address && address.port || options.port;

        // send消息通过lib/internal/cluster/utils.js模块转化为内部消息
        send(message);
      });
    };

    // node@8.3.0 lib/internal/cluster/master.js
    function onmessage(message, handle) {
      const worker = this;

      // ...
      
      else if (message.act === 'listening')
        listening(worker, message);
      
      // ...
    }

    function listening(worker, message) {
      const info = {
        addressType: message.addressType,
        address: message.address,
        port: message.port,
        fd: message.fd
      };

      worker.state = 'listening';
      worker.emit('listening', info);
      cluster.emit('listening', worker, info);
    }

### 服务间的通信

nodejs/cluster模块实现中，主进程内部创建的tcp服务，当其有"connection"事件触发时（即客户端发起请求时），将执行RoundRobinHandle实例的distribute方法，取出空闲的工作进程，并促使该进程发送{act:'newconn'}内部消息。该消息在lib/internal/cluster/child.js模块中得到订阅。工作进程监听到该消息时，将通过onconnection函数执行该进程内的http或https服务的onconnection方法，传入的次参为主进程tcp服务所提供的net#Server实例（即主进程tcp.onconnenction方法的次参），为工作进程中的http或https服务建立新的tcp流;与此同时，工作进程将发送{ack:message.seq}内部消息，执行reply=>{}回调，以调用主进程tcp服务net#Server实例的close方法，促使该服务不再接受新的connection。

    // node@8.3.0 lib/internal/cluster/round_robin_handle.js
    function RoundRobinHandle(key, address, port, addressType, fd) {

      // ...

      this.server.once('listening', () => {
        this.handle = this.server._handle;// lib/net.js模块中构建的Tcp实例

        // 监听"connection"事件
        this.handle.onconnection = (err, handle) => this.distribute(err, handle);
        this.server._handle = null;
        this.server = null;
      });
    }

    RoundRobinHandle.prototype.distribute = function(err, handle) {
      this.handles.push(handle);
      const worker = this.free.shift();

      if (worker)
        this.handoff(worker);
    };

    RoundRobinHandle.prototype.handoff = function(worker) {
      if (worker.id in this.all === false) {
        return;  // Worker is closing (or has closed) the server.
      }

      const handle = this.handles.shift();

      if (handle === undefined) {
        this.free.push(worker);  // Add to ready queue again.
        return;
      }

      const message = { act: 'newconn', key: this.key };

      // 尾参作为工作进程发送{ack:message.seq}内部消息时待执行的函数
      sendHelper(worker.process, message, handle, (reply) => {
        // 工作进程启动http或https服务时，关停tcp服务"connection"事件发出的net#Server
        if (reply.accepted)
          handle.close();
        else
          this.distribute(0, handle);  // Worker is shutting down. Send to another.

        // 嵌套调用handoff方法，若主进程tcp服务没有新的请求，将worker添加this.free中
        this.handoff(worker);
      });
    };

    // node@8.3.0 lib/internal/cluster/child.js
    cluster._setupWorker = function() {
      
      // ...

      function onmessage(message, handle) {
        if (message.act === 'newconn')
          onconnection(message, handle);
        else if (message.act === 'disconnect')
          _disconnect.call(worker, true);
      }
    };

    // Round-robin connection.
    function onconnection(message, handle) {
      const key = message.key;
      const server = handles[key];
      const accepted = server !== undefined;

      send({ ack: message.seq, accepted });

      if (accepted)
        server.onconnection(0, handle);
    }

### 简易流程回顾

#### cluster.fork过程

1. cluster.fork工作进程执行脚本

2. 工作进程执行httpServer.listen

3. 转交listenInCluster函数处理，执行cluster._getServer
    
    - 工作进程发送{act:"queryServer"}，由订阅该消息触发执行queryServer函数，将rr函数添加到回调队列中
    
    - 实例化RoundRobinHandle，创建内部tcp服务
      
      - 调用RoundRobinHandle实例的add方法
      
      - 工作进程发送{ack:message.seq}内部消息，由订阅该消息触发执行rr函数，构建handle={listen,close}对象
      
      - 内部tcp服务监听listening事件，将connection交给工作进程转发给httpServer
    
    - 执行listenInCluster函数中的子函数listenOnMasterHandle
      
      - 为工作进程启动的httpServer添加_handle属性，由于_handle属性为真，置空httpServer._listen2

#### 请求处理过程

1. 接受新的请求，内部tcp服务调用tcpServer._handle.onconnection方法

2. 调用RoundRobinHandle实例的distribute方法，取出空闲的进程

3. 调用RoundRobinHandle实例的handoff方法，工作进程发送{act:'newconn'}内部消息，将handoff方法添加回调队列中

4. 执行订阅{act:'newconn'}内部消息的绑定函数onconnection

5. 工作进程的httpServer服务根据主进程的net#Server实例，发起新的tcp流，处理请求

6. 工作进程发送{ack:message.seq}内部消息，由订阅该消息触发执行handoff方法，调用net#Server实例close方法，不在接受新的请求

7. 嵌套调用handoff方法，将工作进程置为空闲

### 参考案列，来自《[解读Nodejs多核处理模块cluster](http://blog.fens.me/nodejs-core-cluster/)》

    var cluster = require('cluster');
    var http = require('http');
    var numCPUs = require('os').cpus().length;

    if (cluster.isMaster) {
      console.log("master start...");

      // Fork workers.
      for (var i = 0; i < numCPUs; i++) {
        cluster.fork();
      }

      cluster.on('listening',function(worker,address){
        console.log('listening: worker ' + worker.process.pid +', Address: '+address.address+":"+address.port);
      });

      cluster.on('exit', function(worker, code, signal) {
        console.log('worker ' + worker.process.pid + ' died');
      });
    } else {
      http.createServer(function(req, res) {
        console.log('worker'+cluster.worker.id);
        res.writeHead(200);
        res.end("hello world\n");
      }).listen(8080,"127.0.0.1");
    }

### 可借鉴处

复杂事件系统的设计，可由send发送的消息args影响订阅函数on的职能。

按nodejs/cluster模块，send函数的次参将作为钩子函数缓存起来，等到发送{ack}类消息时，再取出相应的回调函数执行；send函数发送的消息对象若含有cmd属性，且前缀为"NODE_"，将视为内部消息。
