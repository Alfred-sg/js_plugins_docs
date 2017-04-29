# redux-saga 0.14.6
[文档地址](http://leonshi.com/redux-saga-in-chinese/index.html)

## 概论

redux-saga包用于监听redux派发action事件，以生成器函数的书写方式触发一系列程式执行。

当特定action被派发dispatch时，redux-saga机制借助proc、io模块执行task任务流。该task任务流允许借助io.take方法挂起task任务流，当特定action被派发时予以执行后续流程；io.put方法派发action；io.race同步触发多个延迟函数，以先执行完成的延迟函数构建返回对象；io.call/apply/cps执行同步或异步函数（返回值为延迟对象promise）；io.fork创建子task任务流；io.join获取子task的返回值；io.select收集state等。

redux-saga的实现，在redux中间件中使用sagaEmitter.emit(action)方法触发事件，通过sagaEmitter.subscribe添加绑定函数。具体实现为，redux-saga借助sagaMiddleware.run(saga)方法将saga封装为一个task任务流，task任务流为生成器书写形式，首次调用next方法为初始化，剩余next方法通过io.take方法或sagaHelpers工具函数添加为sagaEmitter.subscribe的绑定函数，以实现redux派发action时触发task后续流程的执行。

## 基本使用

### 创建saga

#### io.take方式创建saga

	import { take, put } from 'redux-saga/effects'
	
	function* incrementAsync() {
		
	  while(true) {
	
		// 由io.take挂起task任务流，直到redux派发INCREMENT_ASYNC action时触发后续流程执行
		const nextAction = yield take(INCREMENT_ASYNC)
		
		// 执行延迟函数，返回promise对象
		yield delay(1000)
		
		// 派发 INCREMENT_COUNTER
		yield put({type:"INCREMENT_COUNTER"})
	 }
	
	}

	export default [incrementAsync]

#### sagaHelpers方式创建saga

	import { put, call } from 'redux-saga/effects'
	import { delay, takeEvery } from 'redux-saga'
	
	export function* incrementAsync() {
		
	  yield call(delay, 1000)
	  yield put({type: 'INCREMENT'})
		
	}

	export default function* rootSaga() {
	
	  // 由sagaHelpers.takeEvery挂起task任务流，直到redux派发INCREMENT_ASYNC action时触发后续流程执行
	  
	  yield takeEvery('INCREMENT_ASYNC', incrementAsync)
	}

### redux中间件挂载redux-saga

	import sagaMiddleware from 'redux-saga'
	import sagas from '../sagas'
	
	export default function configureStore(initialState) {
	
	  // Note: passing middleware as the last argument to createStore requires redux@>=3.1.0
	 let store = createStore(
	    reducer,
	    initialState,
	    applyMiddleware(/* other middleware, */sagaMiddleware())
	  }
	  
	  sagaMiddleware.run(sagas)
	  
	  return store
	}
	
## 核心源码

### middleware.js挂载redux中间件，添加saga

	'use strict';
	
	Object.defineProperty(exports, "__esModule", {
	  value: true
	});
	exports.default = sagaMiddlewareFactory;
	
	var _utils = require('./utils');
	
	var _proc = require('./proc');
	var _proc2 = _interopRequireDefault(_proc);
	
	var _channel = require('./channel');
	
	function _interopRequireDefault(obj) { return obj && obj.__esModule ? obj : { default: obj }; }
	
	function _toConsumableArray(arr) { 
	  if (Array.isArray(arr)) { 
	    for (var i = 0, arr2 = Array(arr.length); i < arr.length; i++) { 
	      arr2[i] = arr[i]; 
	    } 
	    return arr2; 
	  } else { 
	    return Array.from(arr); 
	  } 
	}
	
	// sagaMiddlewareFactory(
	//    options={
	//      sagaMonitor:{
	//        effectTriggered,// 加载saga或派发action获得effect时调用执行，可用于打印日志
	//        effectResolved,effectRejected,
	//        effectCancelled,// 加载的某个saga，其next方法执行到TASK_CANCEL参数时触发调用，可用于打印日志
	//        actionDispatched:(action)=>{}// action由redux机制派发时触发执行的函数
	//      },
	//      logger:()=>{},
	//      onerror:()=>{},
	//      用于装饰sagaEmitter.emit触发订阅者函数，action为传递给各订阅者函数的参数，即redux机制下的action
	//      emitter:(emit)=>{return (action)=>{console.log(action);emit(action)}},
	//    }
	// )
	function sagaMiddlewareFactory() {
	  var options = arguments.length > 0 && arguments[0] !== undefined ? arguments[0] : {};
	
	  var runSagaDynamically = void 0;
	  var sagaMonitor = options.sagaMonitor;
	
	  // monitors are expected to have a certain interface, let's fill-in any missing ones
	
	  if (sagaMonitor) {
	    sagaMonitor.effectTriggered = sagaMonitor.effectTriggered || _utils.noop;
	    sagaMonitor.effectResolved = sagaMonitor.effectResolved || _utils.noop;
	    sagaMonitor.effectRejected = sagaMonitor.effectRejected || _utils.noop;
	    sagaMonitor.effectCancelled = sagaMonitor.effectCancelled || _utils.noop;
	    sagaMonitor.actionDispatched = sagaMonitor.actionDispatched || _utils.noop;
	  }
	
	  if (_utils.is.func(options)) {
	    if (process.env.NODE_ENV === 'production') {
	      throw new Error('Saga middleware no longer accept Generator functions. Use sagaMiddleware.run instead');
	    } else {
	      throw new Error('You passed a function to the Saga middleware. You are likely trying to start a ' 
	        + '       Saga by directly passing it to the middleware.' 
	        + ' This is no longer possible starting from 0.10.0.' 
	        + '        To run a Saga, you must do it dynamically AFTER mounting the middleware into the store.\n        Example:\n          import createSagaMiddleware from \'redux-saga\'\n          ... other imports\n\n          const sagaMiddleware = createSagaMiddleware()\n          const store = createStore(reducer, applyMiddleware(sagaMiddleware))\n          sagaMiddleware.run(saga, ...args)\n      ');
	    }
	  }
	
	  if (options.logger && !_utils.is.func(options.logger)) {
	    throw new Error('`options.logger` passed to the Saga middleware is not a function!');
	  }
	
	  if (options.onerror) {
	    if (_utils.isDev) (0, _utils.log)('warn', '`options.onerror` is deprecated. Use `options.onError` instead.');
	    options.onError = options.onerror;
	    delete options.onerror;
	  }
	
	  if (options.onError && !_utils.is.func(options.onError)) {
	    throw new Error('`options.onError` passed to the Saga middleware is not a function!');
	  }
	
	  if (options.emitter && !_utils.is.func(options.emitter)) {
	    throw new Error('`options.emitter` passed to the Saga middleware is not a function!');
	  }
	
	  // 由redux中间件机制传入dispatch派发action方法、和getState获取state方法
	  function sagaMiddleware(_ref) {
	    var getState = _ref.getState,
	        dispatch = _ref.dispatch;
	
	    runSagaDynamically = runSaga;
	    // channel.emitter返回{subscribe,emit}用于添加或触发订阅者函数
	    var sagaEmitter = (0, _channel.emitter)();
	    sagaEmitter.emit = (options.emitter || _utils.ident)(sagaEmitter.emit);// 装饰sagaEmitter.emit
	    // 将dispatch获得的参数action更迭为object.assign(action,{'@@redux-saga/SAGA_ACTION':true})
	    var sagaDispatch = (0, _utils.wrapSagaDispatch)(dispatch);
	
	    // 添加saga，saga书写时需用io.take将saga.next后续流程挂起，使用sagaEmitter.emit触发
	    // 挂起操作也可以借助sagaHelper工具集中各工具函数实现
	    function runSaga(saga, args, sagaId) {
	      // sagaEmitter.subscribe添加订阅者函数，sagaDispatch派发action，getState获取state
	      // options为redux-saga的middleware获得的配置项，sagaId第几个saga
	      return (0, _proc2.default)(
	        saga.apply(undefined, _toConsumableArray(args)), 
	        sagaEmitter.subscribe, sagaDispatch, getState, options, sagaId, saga.name);
	    }
	
	    return function (next) {
	      return function (action) {
	        if (sagaMonitor) {
	          sagaMonitor.actionDispatched(action);
	        }
	        // 交由下一个redux中间件处理
	        var result = next(action); // hit reducers
	        // 将action交由sagaEmitter.subscribe方法添加的订阅者函数处理
	        sagaEmitter.emit(action);
	        return result;
	      };
	    };
	  }
	
	  sagaMiddleware.run = function (saga) {
	    for (var _len = arguments.length, args = Array(_len > 1 ? _len - 1 : 0), _key = 1; _key < _len; _key++) {
	      args[_key - 1] = arguments[_key];
	    }
	
	    (0, _utils.check)(runSagaDynamically, _utils.is.notUndef, 'Before running a Saga, you must mount the Saga middleware on the Store using applyMiddleware');
	    (0, _utils.check)(saga, _utils.is.func, 'sagaMiddleware.run(saga, ...args): saga argument must be a Generator function!');
	
	    // 标识符，第几个saga
	    var effectId = (0, _utils.uid)();
	    if (sagaMonitor) {
	      sagaMonitor.effectTriggered({ effectId: effectId, root: true, parentEffectId: 0, effect: { root: true, saga: saga, args: args } });
	    }
	    var task = runSagaDynamically(saga, args, effectId);
	    if (sagaMonitor) {
	      sagaMonitor.effectResolved(effectId, task);
	    }
	    return task;
	  };
	
	  return sagaMiddleware;
	}

### proc.js支配saga的运作流程
	
	'use strict';
	
	Object.defineProperty(exports, "__esModule", {
	  value: true
	});
	exports.TASK_CANCEL = exports.CHANNEL_END = exports.NOT_ITERATOR_ERROR = undefined;
	
	var _typeof = typeof Symbol === "function" && typeof Symbol.iterator === "symbol" ? 
	  function (obj) { return typeof obj; } : 
	  function (obj) { 
	    return obj && typeof Symbol === "function" && obj.constructor === Symbol && 
	    obj !== Symbol.prototype ? "symbol" : typeof obj; 
	  };
	
	exports.default = proc;
	
	var _utils = require('./utils');
	
	var _scheduler = require('./scheduler');
	
	var _io = require('./io');
	
	var _channel = require('./channel');
	
	var _buffers = require('./buffers');
	
	// 设定属性描述符
	function _defineEnumerableProperties(obj, descs) { 
	  for (var key in descs) { 
	    var desc = descs[key]; 
	    desc.configurable = desc.enumerable = true; 
	    if ("value" in desc) desc.writable = true; 
	    Object.defineProperty(obj, key, desc); 
	  } 
	  return obj; 
	}
	
	function _toConsumableArray(arr) { 
	  if (Array.isArray(arr)) { 
	    for (var i = 0, arr2 = Array(arr.length); i < arr.length; i++) { 
	      arr2[i] = arr[i]; 
	    } 
	    return arr2; 
	  } else { 
	    return Array.from(arr); 
	  } 
	}
	
	function _defineProperty(obj, key, value) { 
	  if (key in obj) { 
	    Object.defineProperty(obj, key, { 
	      value: value, 
	      enumerable: true, 
	      configurable: true, 
	      writable: true 
	    }); 
	  } else { 
	    obj[key] = value; 
	  } 
	  return obj; 
	}
	
	var NOT_ITERATOR_ERROR = exports.NOT_ITERATOR_ERROR = 'proc first argument (Saga function result) must be an iterator';
	
	var CHANNEL_END = exports.CHANNEL_END = {
	  toString: function toString() {
	    return '@@redux-saga/CHANNEL_END';
	  }
	};
	var TASK_CANCEL = exports.TASK_CANCEL = {
	  toString: function toString() {
	    return '@@redux-saga/TASK_CANCEL';
	  }
	};
	
	var matchers = {
	  wildcard: function wildcard() {
	    return _utils.kTrue;
	  },
	  default: function _default(pattern) {
	    return function (input) {
	      return input.type === ((typeof pattern === 'undefined' ? 'undefined' 
	        : _typeof(pattern)) === 'symbol' ? pattern : String(pattern));
	    };
	  },
	  array: function array(patterns) {
	    return function (input) {
	      return patterns.some(function (p) {
	        return matcher(p)(input);
	      });
	    };
	  },
	  predicate: function predicate(_predicate) {
	    return function (input) {
	      return _predicate(input);
	    };
	  }
	};
	
	// 构建匹配函数；判断是否通配符无条件，或数组的某一项，或满足某类型，或匹配某函数
	function matcher(pattern) {
	  return (pattern === '*' ? matchers.wildcard : 
	    _utils.is.array(pattern) ? matchers.array : 
	      _utils.is.stringableFunc(pattern) ? matchers.default : 
	      _utils.is.func(pattern) ? matchers.predicate : matchers.default)(pattern);
	}
	
	// 返回{addTask,cancelAll,abort,getTasks,taskNames}用于添加task函数
	// 报错时清空tasks函数集合，执行cb(err,true)；成功时执行cb(res)，mainTask.cont对res不作处理
	//    mainTask的意义：proc设计的迭代流程控制函数next执行到TASK_CANCEL或CHANNEL_END时，跳回到mainTask.cont处理最终响应res
	//        next执行过程报错时，也跳回到mainTask.cont处理错误err
	//        通过mainTask.cont调用end函数，最终启用saga._deferredEnd.resolve|reject处理成功或失败状态
	function forkQueue(name, mainTask, cb) {
	  var tasks = [],
	      result = void 0,
	      completed = false;
	  // 为mainTask添加cont方法，mainTask.cont在成功时将res顺序传入cb回调；报错时执行cb(err,true)，清空tasks
	  addTask(mainTask);
	
	  // completed为真返回；否则，各task函数重置cont方法、并执行cancel方法，清空tasks中缓存的各task函数
	  // 最终调用cb处理err，cb次参传入true
	  function abort(err) {
	    cancelAll();
	    cb(err, true);
	  }
	
	  // 添加task(task函数由proc函数的返回值构成)，为task函数添加cont方法
	  function addTask(task) {
	    tasks.push(task);
	    // completed为真返回；否则，报错时清空tasks，调用cb(res,true)，或者调用cb处理res
	    task.cont = function (res, isErr) {
	      if (completed) {
	        return;
	      }
	
	      (0, _utils.remove)(tasks, task);
	      task.cont = _utils.noop;
	      if (isErr) {
	        // 各task函数重置cont方法、并执行cancel方法，清空tasks中缓存的各task函数；最终调用cb(res,true)
	        abort(res);
	      } else {
	        if (task === mainTask) {
	          result = res;
	        }
	        if (!tasks.length) {
	          completed = true;
	          cb(result);
	        }
	      }
	    };
	  }
	
	  // completed为真返回；否则，各task函数重置cont方法、并执行cancel方法，清空tasks中缓存的各task函数
	  function cancelAll() {
	    if (completed) {
	      return;
	    }
	    completed = true;
	    tasks.forEach(function (t) {
	      t.cont = _utils.noop;
	      t.cancel();
	    });
	    tasks = [];
	  }
	
	  return {
	    addTask: addTask,
	    cancelAll: cancelAll,
	    abort: abort,
	    getTasks: function getTasks() {
	      return tasks;
	    },
	    taskNames: function taskNames() {
	      return tasks.map(function (t) {
	        return t.name;
	      });
	    }
	  };
	}
	
	// 构建迭代器
	function createTaskIterator(_ref) {
	  var context = _ref.context,
	      fn = _ref.fn,
	      args = _ref.args;
	
	  if (_utils.is.iterator(fn)) {
	    return fn;
	  }
	
	  var result = void 0,
	      error = void 0;
	  try {
	    result = fn.apply(context, args);
	  } catch (err) {
	    error = err;
	  }
	
	  if (_utils.is.iterator(result)) {
	    return result;
	  }
	
	  return error ? (0, _utils.makeIterator)(function () {
	    throw error;
	  }) : (0, _utils.makeIterator)(function () {
	    var pc = void 0;
	    var eff = { done: false, value: result };
	    var ret = function ret(value) {
	      return { done: true, value: value };
	    };
	    return function (arg) {
	      if (!pc) {
	        pc = true;
	        return eff;
	      } else {
	        return ret(arg);
	      }
	    };
	  }());
	}
	
	var wrapHelper = function wrapHelper(helper) {
	  return { fn: helper };
	};
	
	// proc(iterator,subscribe,dispatch,getState,options,parentEffectId,name)
	// 参数subscribe即sagaEmitter.subscribe，添加订阅者函数，sagaDispatch派发action，getState获取state
	//    sagaEmitter.emit方法在订阅者函数的回调cb执行过程中，间接执行takers缓存的各获取者函数
	function proc(iterator) {
	  var subscribe = arguments.length > 1 && arguments[1] !== undefined ? arguments[1] : function () {
	    return _utils.noop;
	  };
	  var dispatch = arguments.length > 2 && arguments[2] !== undefined ? arguments[2] : _utils.noop;
	  var getState = arguments.length > 3 && arguments[3] !== undefined ? arguments[3] : _utils.noop;
	  var options = arguments.length > 4 && arguments[4] !== undefined ? arguments[4] : {};
	  var parentEffectId = arguments.length > 5 && arguments[5] !== undefined ? arguments[5] : 0;
	  var name = arguments.length > 6 && arguments[6] !== undefined ? arguments[6] : 'anonymous';
	  var cont = arguments[7];
	
	  (0, _utils.check)(iterator, _utils.is.iterator, NOT_ITERATOR_ERROR);
	
	  var sagaMonitor = options.sagaMonitor,
	      logger = options.logger,
	      onError = options.onError;
	
	  var log = logger || _utils.log;
	  // channel.stdChannel(subscribe) 返回{take,flush,close}
	  // take(cb,matcher)方法缓存taker获取者函数cb，并向cb添加'@@redux-saga/MATCH'属性为次参matcher
	  // flush方法无操作，close方法关闭chan，并清空订阅者函数
	  // sagaEmitter.emit方法在订阅者函数的回调cb执行过程中，间接执行takers缓存的各获取者函数
	  var stdChannel = (0, _channel.stdChannel)(subscribe);
	
	  // 由各runForkEffect等函数进行赋值，流程终止时调用
	  next.cancel = _utils.noop;
	
	  // newTask函数返回
	  // {
	  //   '@@redux-saga/TASK': true,
	  //   id: parentEffectId,
	  //   name: name,
	  //   cont: cont,
	  //   joiners: [],
	  //   cancel: cancel,
	  //   isRunning: ()=>iterator._isRunning,
	  //   isCancelled: ()=>iterator._isCancelled,
	  //   isAborted: ()=>iterator._isAborted,
	  //   result: ()=>iterator._result,
	  //   error: ()=>iterator._error,
	  //   get done 属性修饰符约定只能获取，返回promise对象
	  // }
	  var task = newTask(parentEffectId, name, iterator, cont);
	
	  // proc设计的迭代流程控制函数next执行到TASK_CANCEL或CHANNEL_END时，跳回到mainTask.cont处理最终响应res
	  // next执行过程报错时，也跳回到mainTask.cont处理错误err
	  // 通过mainTask.cont调用end函数，最终启用saga._deferredEnd.resolve|reject处理成功或失败状态
	  var mainTask = { name: name, cancel: cancelMain, isRunning: true };
	
	  // forkQueue(name,mainTask,cb)返回{addTask,cancelAll,abort,getTasks,taskNames}用于添加task函数
	  // 报错时清空tasks函数集合，执行cb(err,true)；成功时执行cb(res)，mainTask.cont对res不作处理
	  // forkQueue函数为mainTask添加cont方法，mainTask.cont在成功时将res顺序传入cb回调；报错时执行cb(err,true)，清空tasks
	  var taskQueue = forkQueue(name, mainTask, end);
	
	  function cancelMain() {
	    if (mainTask.isRunning && !mainTask.isCancelled) {
	      mainTask.isCancelled = true;
	      next(TASK_CANCEL);
	    }
	  }
	
	  function cancel() {
	    if (iterator._isRunning && !iterator._isCancelled) {
	      iterator._isCancelled = true;
	      taskQueue.cancelAll();
	
	      end(TASK_CANCEL);
	    }
	  }
	
	  cont && (cont.cancel = cancel);
	
	  // 标记iterator即saga的_isRunning为true，iterator也可能是子task的iterator
	  // mainTask在整个生命周期中存活，子task自在调用时存活
	  iterator._isRunning = true;
	
	  // 执行saga；或当saga为sagaHelper中工具函数创建或调用了io.take方法，将saga.next方法挂起，由middleware模块派发action时触发
	  next();
	
	  return task;
	
	  // 初始化执行saga.next
	  // 或者当saga为sagaHelper中工具函数创建或调用了io.take方法，将saga.next方法挂起，由middleware模块派发action时触发执行
	  function next(arg, isErr) {
	    if (!mainTask.isRunning) {
	      throw new Error('Trying to resume an already finished generator');
	    }
	
	    try {
	      var result = void 0;
	      if (isErr) {
	        result = iterator.throw(arg);
	      } else if (arg === TASK_CANCEL) {
	
	        mainTask.isCancelled = true;
	        
	        // 由各runForkEffect等函数进行赋值，流程终止时调用
	        next.cancel();
	   
	        result = _utils.is.func(iterator.return) ? iterator.return(TASK_CANCEL) : { done: true, value: TASK_CANCEL };
	      } else if (arg === CHANNEL_END) {
	        result = _utils.is.func(iterator.return) ? iterator.return() : { done: true };
	      } else {
	        result = iterator.next(arg);
	      }
	
	      if (!result.done) {
	        runEffect(result.value, parentEffectId, '', next);
	      } else {
	        // proc设计的迭代流程控制函数next执行到TASK_CANCEL或CHANNEL_END时，表示迭代终结
	        // 调用mainTask处理最终返回值result.value
	        mainTask.isMainRunning = false;
	        mainTask.cont && mainTask.cont(result.value);
	      }
	    } catch (error) {
	      if (mainTask.isCancelled) {
	        log('error', 'uncaught at ' + name, error.message);
	      }
	      mainTask.isMainRunning = false;
	
	      // 清空forkQueue中tasks函数集合，并将error顺序传递给end函数，并执行end(error,true)
	      mainTask.cont(error, true);
	    }
	  }
	
	  // iterator即为saga，调用saga._deferredEnd.resolve处理成功时的响应，saga._deferredEnd.reject处理错误
	  // task.cont不存在时，执行钩子middleware模块sagaMiddlewareFactory的参数option.onError方法
	  // 成功或失败时，都执行task.cont方法、task.joiners函数集合并清空task.joiners函数集合
	  // 业务上，end函数作为forkQueue函数的尾参，
	  function end(result, isErr) {
	    iterator._isRunning = false;
	
	    // 取出stdChannel函数中takers缓存中的各获取者函数，以参数{type:'@@redux-saga/CHANNEL_END'}逐个执行
	    // 并清空sagaEmitter中的订阅者函数
	    stdChannel.close();
	
	    if (!isErr) {
	      if (result === TASK_CANCEL && _utils.isDev) {
	        log('info', name + ' has been cancelled', '');
	      }
	      iterator._result = result;
	      iterator._deferredEnd && iterator._deferredEnd.resolve(result);
	    } else {
	      if (result instanceof Error) {
	        result.sagaStack = 'at ' + name + ' \n ' + (result.sagaStack || result.stack);
	      }
	      if (!task.cont) {
	        log('error', 'uncaught', result.sagaStack || result.stack);
	        if (result instanceof Error && onError) {
	          onError(result);
	        }
	      }
	      iterator._error = result;
	      iterator._isAborted = true;
	      iterator._deferredEnd && iterator._deferredEnd.reject(result);
	    }
	    task.cont && task.cont(result, isErr);
	    task.joiners.forEach(function (j) {
	      return j.cb(result, isErr);
	    });
	    task.joiners = null;
	  }
	
	  // proc的流程控制函数next接受传参不是TASK_CANCEL或CHANNEL_END时，调用runEffect函数处理saga.next(arg)返回值effect
	  //    根据effect的类型控制不同的处理逻辑，如effect为{IO:true,TAKE:{}}形式时，调用runTakeEffect函数作处理
	  // runEffect(effect,parentEffectId,label,cb)
	  // 参数effect为saga.next(arg)返回值，cb为next函数执行saga.next或处理最终返回值或错误
	  function runEffect(effect, parentEffectId) {
	    var label = arguments.length > 2 && arguments[2] !== undefined ? arguments[2] : '';
	    var cb = arguments[3];
	
	    var effectId = (0, _utils.uid)();
	    sagaMonitor && sagaMonitor.effectTriggered({ effectId: effectId, parentEffectId: parentEffectId, label: label, effect: effect });
	
	    var effectSettled = void 0;
	
	    // 通过currCb调用next函数，接着执行interator.next即saga.next方法
	    function currCb(res, isErr) {
	      if (effectSettled) {
	        return;
	      }
	
	      effectSettled = true;
	      cb.cancel = _utils.noop;
	      if (sagaMonitor) {
	        isErr ? sagaMonitor.effectRejected(effectId, res) : sagaMonitor.effectResolved(effectId, res);
	      }
	
	      // cb即proc下next流程控制函数
	      cb(res, isErr);
	    }
	
	    // currCb.cancel更迭为interator.next即saga.next的再次执行后赋值
	    // interator.next即saga.next得到参数为TASK_CANCEL时，通过next.cancel调用currCb.cancel方法
	    currCb.cancel = _utils.noop;
	
	    // interator.next即saga.next方法执行的参数为TASK_CANCEL，终止当前的saga；cb即next函数
	    cb.cancel = function () {
	      // currCb执行后，effectSettled设置为true，互斥关系
	      if (effectSettled) {
	        return;
	      }
	
	      effectSettled = true;
	
	      try {
	        currCb.cancel();
	      } catch (err) {
	        log('error', 'uncaught at ' + name, err.message);
	      }
	      currCb.cancel = _utils.noop; 
	
	      sagaMonitor && sagaMonitor.effectCancelled(effectId);
	    };
	
	    var data = void 0;
	    return (
	      // interator.next即saga.next方法执行时返回promise延迟对象，将currCb作为promise对象的回调并执行
	      _utils.is.promise(effect) ? resolvePromise(effect, currCb) : 
	
	      // interator.next即saga.next方法执行时返回sagaHelpers构建的迭代器，构建子task
	      // effect为调用sagaHelpers.takeEveryHelper | takeLatestHelper | throttleHelper 方法构建的迭代器
	      // 通常proc函数的入参intertor即saga就是sagaHelpers.takeEveryHelper等方法构建的迭代器{name,next,throw,return}
	      _utils.is.helper(effect) ? runForkEffect(wrapHelper(effect), effectId, currCb) : 
	
	      // interator.next即saga.next方法执行时返回迭代器，递归调用proc函数构建子task
	      _utils.is.iterator(effect) ? resolveIterator(effect, effectId, name, currCb) : 
	
	      // interator.next即saga.next方法执行时返回数组，对数组中每个子项迭代调用runEffect函数
	      // 每个runEffect函数的返回值串接成数组，该数组作为currCb的参数
	      _utils.is.array(effect) ? runParallelEffect(effect, effectId, currCb) : 
	
	      // effect为{IO:true,TAKE:{}}形式，io.asEffect.take获取effect的TAKE属性{channel}或{pattern}
	      //    该情形在interator.next即saga.next中调用io.take方法时触发
	      //    或者当saga为sagaHelpers.takeEveryHelper|takeLatestHelper|throttleHelper方法构建的迭代器
	      // 意义是挂起currCb；当redux派发action时，再调用currCb处理saga中的后续流程
	      //    redux派发的action须匹配io.take方法返回值pattern时，将action作为interator.next(arg)的参数arg
	      _utils.is.notUndef(data = _io.asEffect.take(effect)) ? runTakeEffect(data, currCb) : 
	
	      // effect为{ [IO]:true, PUT:{channel,action} }形式，io.asEffect.put获取effect的PUT属性{channel,action}
	      //    该情形在interator.next即saga.next中调用io.put方法时触发
	      // 意义是用于dispatch一个action，返回值可作为延迟对象或一般返回值交给currCb处理
	      _utils.is.notUndef(data = _io.asEffect.put(effect)) ?  runPutEffect(data, currCb) : 
	
	      // effect为{ [IO]:true, RACE:effects }形式，io.asEffect.race获取effect的RACE属性
	      //    该情形在interator.next即saga.next中调用io.race方法时触发
	      // 意义是多个延迟函数予以竞赛，io.race({effect1:fetch1,effect2:fetch2})，先执行完成的延迟函数构成返回值
	      //    如fetch1函数先执行完成，返回{effect1:res1,effect2:undefined}，作为currCb的参数
	      _utils.is.notUndef(data = _io.asEffect.race(effect)) ? runRaceEffect(data, effectId, currCb) : 
	
	      // effect为{ [IO]:true, CALL:{context,fn,args} }，io.asEffect.call获取effect的CALL属性
	      //    该情形在saga中调用io.call或io.apply方法时触发，以上下文context执行CALL属性中的函数fn
	      //    io.call([context,fn] | {context,fn} | fn,...args)与apply(context,fn,[args])设置上下文方式不同
	      // 意义是执行CALL属性中的函数fn，其返回值作为延迟对象或一般返回值交给currCb处理
	      _utils.is.notUndef(data = _io.asEffect.call(effect)) ? runCallEffect(data, effectId, currCb) : 
	
	      // effect为{ [IO]:true, CPS:{context,fn,args} }形式，io.asEffect.cps获取effect的CPS属性
	      //    该情形在interator.next即saga.next中调用io.cps方法时触发，以上下文context执行CALL属性中的函数fn
	      // 意义是执行CPS属性中的函数fn，currCb作为尾参数，在函数fn中以回调函数方式执行；典型的fn函数为node端的readFile函数
	      _utils.is.notUndef(data = _io.asEffect.cps(effect)) ?  runCPSEffect(data, currCb) : 
	
	      // effect为{ [IO]:true, FORK:{context,fn,args} }形式，io.asEffect.fork获取effect的FORK属性
	      //    该情形在interator.next即saga.next中调用io.fork方法时触发，函数fn转化为迭代器后创建子task
	      // 意义是创建子task，由子task处理延迟的action事件；避免由saga处理延迟事件时，saga.next方法被挂起，外部无法调用
	      //    io.fork执行后，saga.next接受的参数为通过函数fn构建的子task
	      // 注：io.fork的函数可接受普通函数、生成器函数或迭代器函数
	      _utils.is.notUndef(data = _io.asEffect.fork(effect)) ? runForkEffect(data, effectId, currCb) : 
	
	      // effect为{ [IO]:true, JOIN:task }形式，io.asEffect.join获取effect的JOIN属性
	      //    该情形在interator.next即saga.next中调用io.join方法时触发，收集子task的返回值
	      // 意义是收集子task的返回值；或者当子task在整个生命周期中存活，则将主task添加到子task的joinners属性中
	      _utils.is.notUndef(data = _io.asEffect.join(effect)) ? runJoinEffect(data, currCb) : 
	
	      // effect为{ [IO]:true, CANCEL:task }形式，io.asEffect.cancel获取effect的CANCEL属性
	      //    该情形在interator.next即saga.next中调用io.cancel方法时触发，终结主或子task
	      // 意义是终结主或子task的运作
	      _utils.is.notUndef(data = _io.asEffect.cancel(effect)) ? runCancelEffect(data, currCb) : 
	
	      // effect为{ [IO]:true, SELECT:{selector,args} }形式，io.asEffect.select获取effect的SELECT属性
	      //    该情形在interator.next即saga.next中调用io.select方法时触发，由参数args、selector获取state，作为saga.next的参数
	      // 意义是由参数args、selector获取state，作为saga.next的参数；可用于特定的state时的日志操作
	      _utils.is.notUndef(data = _io.asEffect.select(effect)) ? runSelectEffect(data, currCb) : 
	
	      // effect为{ [IO]:true, ACTION_CHANNEL:{pattern,buffer} }形式，io.asEffect.actionChannel获取effect的ACTION_CHANNEL属性
	      //    该情形在interator.next即saga.next中调用io.actionChannel方法时触发，意义派发某action时执行特定的订阅函数
	      // 意义是添加订阅函数，监听某action的执行，触发订阅函数的调用
	      _utils.is.notUndef(data = _io.asEffect.actionChannel(effect)) ? runChannelEffect(data, currCb) :
	
	      // effect为{ [IO]:true, FLUSH:channel }形式，io.asEffect.flush获取effect的FLUSH属性
	      //    该情形在interator.next即saga.next中调用io.flush法时触发，触发channel中订阅函数的执行，处理当前的action
	      _utils.is.notUndef(data = _io.asEffect.flush(effect)) ? runFlushEffect(data, currCb) : 
	
	      // effect为{ [IO]:true, CANCELLED:{} }形式，io.asEffect.cancelled获取effect的CANCELLED属性
	      //    该情形在interator.next即saga.next中调用io.cancel方法时触发
	      // 意义是判断主task是否执行完结，saga.next获得参数为主task执行状态
	      _utils.is.notUndef(data = _io.asEffect.cancelled(effect)) ? runCancelledEffect(data, currCb) : /* anything else returned as is        */
	      
	      currCb(effect)
	    );
	  }
	
	  // intertor.next(arg)即saga.next(arg)返回promise对象时，将currCub作为promise的回调
	  // 参数cb为runEffect函数中的currCub，将调用proc下next流程控制函数
	  function resolvePromise(promise, cb) {
	    var cancelPromise = promise[_utils.CANCEL];
	    if (typeof cancelPromise === 'function') {
	      cb.cancel = cancelPromise;
	    }
	    promise.then(cb, function (error) {
	      return cb(error, true);
	    });
	  }
	
	  // intertor.next(arg)即saga.next(arg)返回迭代器对象时，调用proc构建新的子task
	  // 参数cb为runEffect函数中的currCub，将调用proc下next流程控制函数
	  function resolveIterator(iterator, effectId, name, cb) {
	    proc(iterator, subscribe, dispatch, getState, options, effectId, name, cb);
	  }
	
	  // runTakeEffect函数在saga中调用io.take方法时触发调用，处理io.take方法返回值的TAKE属性
	  //      也可能当saga为sagaHelpers.takeEveryHelper|takeLatestHelper|throttleHelper构建的迭代器时触发
	  //   意义是将takecb存入stdChannel中，redux机制派发action时通过sagaEmitter.emit传入action，调用takecb
	  //      takeCb执行过程中，执行saga后续流程
	  // 参数_ref2为io.take方法返回值的TAKE属性{channel}或{pattern}
	  // 参数cb为runEffect函数中的currCub，将调用proc下next流程控制函数
	  function runTakeEffect(_ref2, cb) {
	    var channel = _ref2.channel,
	        pattern = _ref2.pattern,
	        maybe = _ref2.maybe;
	
	    channel = channel || stdChannel;
	    var takeCb = function takeCb(inp) {
	      return inp instanceof Error ? cb(inp, true) : 
	        (0, _channel.isEnd)(inp) && !maybe ? cb(CHANNEL_END) : cb(inp);
	    };
	    try {
	      // 将takecb存入stdChannel中，redux机制派发action时通过sagaEmitter.emit传入action，调用takecb
	      // takeCb的'@@redux-saga/MATCH'属性置为次参matcher(pattern)
	      channel.take(takeCb, matcher(pattern));
	    } catch (err) {
	      return cb(err, true);
	    }
	    cb.cancel = takeCb.cancel;
	  }
	
	  // runPutEffect函数在saga中调用io.put方法时触发调用，处理io.put方法返回值的PUT属性
	  //     意义是在saga中触发redux机制派发action的动作，返回值以延迟函数或普通返回值处理
	  // 参数_ref3为io.put方法返回值的Put属性{channel,action}
	  // 参数cb为runEffect函数中的currCub，将调用proc下next流程控制函数
	  function runPutEffect(_ref3, cb) {
	    var channel = _ref3.channel,
	        action = _ref3.action,
	        resolve = _ref3.resolve;
	
	    (0, _scheduler.asap)(function () {
	      var result = void 0;
	      try {
	        result = (channel ? channel.put : dispatch)(action);
	      } catch (error) {
	        if (channel || resolve) return cb(error, true);
	        log('error', 'uncaught at ' + name, error.stack || error.message || error);
	      }
	
	      if (resolve && _utils.is.promise(result)) {
	        resolvePromise(result, cb);
	      } else {
	        return cb(result);
	      }
	    });
	  }
	
	  // runCallEffect函数在saga中调用io.call或io.apply方法时触发调用，处理io.call或或io.apply方法返回值的CALL属性
	  // 参数_ref4为io.call或io.apply方法返回值的CALL属性{context,fn,args}，执行fn函数
	  // 参数cb为runEffect函数中的currCub，将调用proc下next流程控制函数
	  function runCallEffect(_ref4, effectId, cb) {
	    var context = _ref4.context,
	        fn = _ref4.fn,
	        args = _ref4.args;
	
	    var result = void 0;
	    try {
	      result = fn.apply(context, args);
	    } catch (error) {
	      return cb(error, true);
	    }
	    return _utils.is.promise(result) ? resolvePromise(result, cb) : 
	      _utils.is.iterator(result) ? resolveIterator(result, effectId, fn.name, cb) : cb(result);
	  }
	
	  // runCPSEffect函数在saga中调用io.cps方法时触发调用，处理io.cps方法返回值的CPS属性
	  // 参数_ref5为io.cps方法返回值的CPS属性{context,fn,args}，执行fn函数，并将currCub作为函数fn的尾参回调函数
	  //    典型的fn函数如node的readFile函数，执行过程中自动调用尾参回调函数处理错误或成功值
	  // 参数cb为runEffect函数中的currCub，将调用proc下next流程控制函数
	  function runCPSEffect(_ref5, cb) {
	    var context = _ref5.context,
	        fn = _ref5.fn,
	        args = _ref5.args;
	
	    try {
	      var cpsCb = function cpsCb(err, res) {
	        return _utils.is.undef(err) ? cb(res) : cb(err, true);
	      };
	      fn.apply(context, args.concat(cpsCb));
	      if (cpsCb.cancel) {
	        cb.cancel = function () {
	          return cpsCb.cancel();
	        };
	      }
	    } catch (error) {
	      return cb(error, true);
	    }
	  }
	
	  // 针对异常情况：点击某按钮触发fetchApi延迟函数，最后更新按钮loading状态
	  // watchFetch通过sagaHelper挂载为saga时，一次点击延迟过程中，再次点击无法调用redux-saga机制处理FETCH_POSTS
	  // function* watchFetch() {
	  //   while ( yield take(FETCH_POSTS) ) {
	  //     yield put( actions.requestPosts() )
	  //     const posts = yield call(fetchApi, '/posts') // blocking call
	  //     yield put( actions.receivePosts(posts) )
	  //   }
	  // }
	
	  // io.fork方法处理前述异常情况，在saga.next方法执行时创建子task，由子task处理FETCH_POSTS
	  // 以避免saga.next在延迟函数执行过程中挂起，外部无法调用
	  // function* fetchPosts() {
	  //   yield put( actions.requestPosts() )
	  //   const posts = yield call(fetchApi, '/posts')
	  //   yield put( actions.receivePosts(posts) )
	  // }
	  // function* watchFetch() {
	  //   while ( yield take(FETCH_POSTS) ) {
	  //     yield fork(fetchPosts) // non blocking call
	  //   }
	  // }
	
	  // runForkEffect函数在saga中调用io.fork方法时触发调用，处理io.fork方法返回值的FORK属性
	  // 参数_ref4为io.fork方法返回值的FORK属性{context,fn,args}，通过fn构建迭代器函数，创建子task
	  //    通过middleware.run导入的saga.next执行过程中，外部无法触发currCub的执行，同样的action无法触发redux-saga的处理机制
	  //    这种情况下，在saga.next的执行过程中创建子task，由子task处理同名且为延迟事件的action
	  // 参数cb为runEffect函数中的currCub，将调用proc下next流程控制函数
	  function runForkEffect(_ref6, effectId, cb) {
	    var context = _ref6.context,
	        fn = _ref6.fn,
	        args = _ref6.args,
	        detached = _ref6.detached;
	
	    var taskIterator = createTaskIterator({ context: context, fn: fn, args: args });
	
	    try {
	      (0, _scheduler.suspend)();
	      var _task = proc(taskIterator, subscribe, dispatch, getState, options, effectId, fn.name, detached ? null : _utils.noop);
	
	      if (detached) {
	        cb(_task);
	      } else {
	        if (taskIterator._isRunning) {
	          taskQueue.addTask(_task);
	          cb(_task);
	        } else if (taskIterator._error) {
	          taskQueue.abort(taskIterator._error);
	        } else {
	          cb(_task);
	        }
	      }
	    } finally {
	      (0, _scheduler.flush)();
	    }
	  }
	
	  // function* child() { ... }
	  // function *parent() {
	  //   // non blocking call
	  //   const task = yield fork(subtask, ...args)
	  //   // ... later
	  //   // now a blocking call, will resume with the outcome of task
	  //   const result = yield join(task)
	  // }
	
	  // runJoinEffect函数在saga中调用io.join方法时触发调用，处理io.join方法返回值的TASK属性
	  // 参数t为io.join方法或返回值的TASK属性{task}，即子task，主要目的是获取子task的执行结果
	  // 参数cb为runEffect函数中的currCub，将调用proc下next流程控制函数
	  function runJoinEffect(t, cb) {
	    // t.isRunning()为真，意味子task在整个生命周期中都存活，将主task添加到子task的joinners属性中
	    if (t.isRunning()) {
	      var joiner = { task: task, cb: cb };
	      cb.cancel = function () {
	        return (0, _utils.remove)(t.joiners, joiner);
	      };
	      t.joiners.push(joiner);
	    // 否则收集子task的返回值或错误
	    } else {
	      t.isAborted() ? cb(t.error(), true) : cb(t.result());
	    }
	  }
	
	  // runCancelEffect函数在saga中调用io.cancel方法时触发调用，处理io.cancel方法返回值的CANCEL属性
	  // 参数t为io.cancel方法或返回值的CANCEL属性{task}，即主或子task，主要目的是终结主或子task
	  // 参数cb为runEffect函数中的currCub，将调用proc下next流程控制函数
	  function runCancelEffect(taskToCancel, cb) {
	    if (taskToCancel === _utils.SELF_CANCELLATION) {
	      taskToCancel = task;
	    }
	    if (taskToCancel.isRunning()) {
	      taskToCancel.cancel();
	    }
	    cb();
	  }
	
	  // interator.next即saga.next方法执行时返回数组，对数组中每个子项迭代调用runEffect函数
	  // 参数cb为runEffect函数中的currCub，将调用proc下next流程控制函数，获得的参数为由每个effect处理值构建的数组
	  function runParallelEffect(effects, effectId, cb) {
	    if (!effects.length) {
	      return cb([]);
	    }
	
	    var completedCount = 0;
	    var completed = void 0;
	    var results = Array(effects.length);
	
	    function checkEffectEnd() {
	      if (completedCount === results.length) {
	        completed = true;
	        cb(results);
	      }
	    }
	
	    var childCbs = effects.map(function (eff, idx) {
	      var chCbAtIdx = function chCbAtIdx(res, isErr) {
	        if (completed) {
	          return;
	        }
	        if (isErr || (0, _channel.isEnd)(res) || res === CHANNEL_END || res === TASK_CANCEL) {
	          cb.cancel();
	          cb(res, isErr);
	        } else {
	          results[idx] = res;
	          completedCount++;
	          checkEffectEnd();
	        }
	      };
	      chCbAtIdx.cancel = _utils.noop;
	      return chCbAtIdx;
	    });
	
	    cb.cancel = function () {
	      if (!completed) {
	        completed = true;
	        childCbs.forEach(function (chCb) {
	          return chCb.cancel();
	        });
	      }
	    };
	
	    effects.forEach(function (eff, idx) {
	      return runEffect(eff, effectId, idx, childCbs[idx]);
	    });
	  }
	
	  // const {posts, timeout} = yield race({
	  //   posts   : call(fetchApi, '/posts'),
	  //   timeout : call(delay, 1000)
	  // })
	  // if(posts)
	  //   put( actions.receivePosts(posts) )
	  // else
	  //   put( actions.timeoutError() )
	
	  // runForkEffect函数在saga中调用io.race方法时触发调用，处理io.race方法返回值的RACE属性{effect:fetch(){}}
	  // 参数effects为io.race方法返回值的RACE属性{effect1:fetch1(){},effect2:fetch2(){}}，执行延迟函数fetch1、fetch2
	  //    先执行完成的延迟函数以对象构成返回值，若fetch1先执行完成，则返回{effect1:res1,effect2:undefined}
	  //    返回值作为currCub的参数，并执行currCub，再次调用proc下next流程控制函数
	  // 参数cb为runEffect函数中的currCub，将调用proc下next流程控制函数
	  function runRaceEffect(effects, effectId, cb) {
	    var completed = void 0;
	    var keys = Object.keys(effects);
	    var childCbs = {};
	
	    keys.forEach(function (key) {
	      var chCbAtKey = function chCbAtKey(res, isErr) {
	        if (completed) {
	          return;
	        }
	
	        if (isErr) {
	          cb.cancel();
	          cb(res, true);
	        } else if (!(0, _channel.isEnd)(res) && res !== CHANNEL_END && res !== TASK_CANCEL) {
	          cb.cancel();
	          completed = true;
	          cb(_defineProperty({}, key, res));
	        }
	      };
	      chCbAtKey.cancel = _utils.noop;
	      childCbs[key] = chCbAtKey;
	    });
	
	    cb.cancel = function () {
	      // prevents unnecessary cancellation
	      if (!completed) {
	        completed = true;
	        keys.forEach(function (key) {
	          return childCbs[key].cancel();
	        });
	      }
	    };
	    keys.forEach(function (key) {
	      if (completed) {
	        return;
	      }
	      runEffect(effects[key], effectId, key, childCbs[key]);
	    });
	  }
	
	  // runCancelEffect函数在saga中调用io.select方法时触发调用，处理io.select方法返回值的SELECT属性
	  // 参数t为io.select方法或返回值的SELECT属性{selector,args}，由参数args、selector获取state，作为saga.next的参数
	  // 参数cb为runEffect函数中的currCub，将调用proc下next流程控制函数
	  function runSelectEffect(_ref7, cb) {
	    var selector = _ref7.selector,
	        args = _ref7.args;
	
	    try {
	      var state = selector.apply(undefined, [getState()].concat(_toConsumableArray(args)));
	      cb(state);
	    } catch (error) {
	      cb(error, true);
	    }
	  }
	
	  // runChannelEffect函数在saga中调用io.actionChannel方法时触发调用，处理io.actionChannel方法返回值的ACTION_CHANNEL属性
	  // 参数t为io.actionChannel方法或返回值的ACTION_CHANNEL属性{pattern,buffer}
	  //    pattern作为action的筛选条件，buffer作为channel.eventChannel的缓存
	  //    意义是添加某action派发时触发执行channel.eventChannel下的订阅函数
	  // 参数cb为runEffect函数中的currCub，将调用proc下next流程控制函数
	  function runChannelEffect(_ref8, cb) {
	    var pattern = _ref8.pattern,
	        buffer = _ref8.buffer;
	
	    var match = matcher(pattern);// 构建匹配函数
	    match.pattern = pattern;
	    cb((0, _channel.eventChannel)(subscribe, buffer || _buffers.buffers.fixed(), match));
	  }
	
	  function runCancelledEffect(data, cb) {
	    cb(!!mainTask.isCancelled);
	  }
	
	  // 触发channel中订阅函数的执行
	  function runFlushEffect(channel, cb) {
	    channel.flush(cb);
	  }
	
	  // 返回
	  // {
	  //   '@@redux-saga/TASK': true,
	  //   id: id,
	  //   name: name,
	  //   cont: cont,
	  //   joiners: [],
	  //   cancel: cancel,
	  //   isRunning: ()=>iterator._isRunning,
	  //   isCancelled: ()=>iterator._isCancelled,
	  //   isAborted: ()=>iterator._isAborted,
	  //   result: ()=>iterator._result,
	  //   error: ()=>iterator._error,
	  //   get done 属性修饰符约定只能获取，返回promise对象
	  // }
	  function newTask(id, name, iterator, cont) {
	    var _done, _ref9, _mutatorMap;
	
	    iterator._deferredEnd = null;
	    return _ref9 = {}, 
	      _defineProperty(_ref9, _utils.TASK, true),// utils.TASK 即常量'@@redux-saga/TASK'
	      _defineProperty(_ref9, 'id', id), 
	      _defineProperty(_ref9, 'name', name), 
	      _done = 'done', 
	      _mutatorMap = {}, 
	      _mutatorMap[_done] = _mutatorMap[_done] || {}, 
	      _mutatorMap[_done].get = function () {
	      if (iterator._deferredEnd) {
	          return iterator._deferredEnd.promise;
	        } else {
	          // utils.deferred(props) 构建promise对象，并返回{promise,resolve,reject,...props}
	          var def = (0, _utils.deferred)();
	          iterator._deferredEnd = def;
	          if (!iterator._isRunning) {
	            iterator._error ? def.reject(iterator._error) : def.resolve(iterator._result);
	          }
	          return def.promise;
	        }
	      }, 
	      _defineProperty(_ref9, 'cont', cont), 
	      _defineProperty(_ref9, 'joiners', []), 
	      _defineProperty(_ref9, 'cancel', cancel), 
	      _defineProperty(_ref9, 'isRunning', function isRunning() {
	        return iterator._isRunning;
	      }), 
	      _defineProperty(_ref9, 'isCancelled', function isCancelled() {
	        return iterator._isCancelled;
	      }), 
	      _defineProperty(_ref9, 'isAborted', function isAborted() {
	        return iterator._isAborted;
	      }), 
	      _defineProperty(_ref9, 'result', function result() {
	        return iterator._result;
	      }), 
	      _defineProperty(_ref9, 'error', function error() {
	        return iterator._error;
	      }), 
	      _defineEnumerableProperties(_ref9, _mutatorMap), 
	      _ref9;
	  }
	}

### io.js将saga.next返回值分类，以设置处理策略
	
	'use strict';
	
	Object.defineProperty(exports, "__esModule", {
	  value: true
	});
	exports.asEffect = exports.takem = undefined;
	
	var _slicedToArray = function () { 
	  // 参数arr若为数组，获取长度为i的该数组备份；若为对象且可迭代，获取长度为i的迭代内容，构成数组返回
	  function sliceIterator(arr, i) { 
	    var _arr = []; 
	    var _n = true; 
	    var _d = false; 
	    var _e = undefined; 
	    try { 
	      for (var _i = arr[Symbol.iterator](), _s; !(_n = (_s = _i.next()).done); _n = true) { 
	        _arr.push(_s.value); 
	        if (i && _arr.length === i) break; 
	      } 
	    } catch (err) { 
	      _d = true; 
	      _e = err; 
	    } finally { 
	      try { 
	        if (!_n && _i["return"]) _i["return"](); 
	      } finally { 
	        if (_d) throw _e; 
	      } 
	    } 
	    return _arr; 
	  } 
	  return function (arr, i) { 
	    if (Array.isArray(arr)) { 
	      return arr; 
	    } else if (Symbol.iterator in Object(arr)) { 
	      return sliceIterator(arr, i); 
	    } else { 
	      throw new TypeError("Invalid attempt to destructure non-iterable instance"); 
	    } 
	  }; 
	}();
	
	exports.take = take;
	exports.put = put;
	exports.race = race;
	exports.call = call;
	exports.apply = apply;
	exports.cps = cps;
	exports.fork = fork;
	exports.spawn = spawn;
	exports.join = join;
	exports.cancel = cancel;
	exports.select = select;
	exports.actionChannel = actionChannel;
	exports.cancelled = cancelled;
	exports.flush = flush;
	exports.takeEvery = takeEvery;
	exports.takeLatest = takeLatest;
	exports.throttle = throttle;
	
	var _utils = require('./utils');
	
	var _sagaHelpers = require('./sagaHelpers');
	
	// 设置obj对象的key属性为value，obj对象原有key属性，则key属性可枚举、可配置、可改写
	function _defineProperty(obj, key, value) { 
	  if (key in obj) { 
	    Object.defineProperty(obj, key, { 
	      value: value, enumerable: true, configurable: true, writable: true 
	    }); 
	  } else { 
	    obj[key] = value; 
	  } 
	  return obj; 
	}
	
	var IO = (0, _utils.sym)('IO');
	var TAKE = 'TAKE';
	var PUT = 'PUT';
	var RACE = 'RACE';
	var CALL = 'CALL';
	var CPS = 'CPS';
	var FORK = 'FORK';
	var JOIN = 'JOIN';
	var CANCEL = 'CANCEL';
	var SELECT = 'SELECT';
	var ACTION_CHANNEL = 'ACTION_CHANNEL';
	var CANCELLED = 'CANCELLED';
	var FLUSH = 'FLUSH';
	
	var TEST_HINT = '\n(HINT: if you are getting this errors in tests,' 
	  + ' consider using createMockTask from redux-saga/utils)';
	
	// 方法deprecated更迭为方法preferred，提示
	var deprecationWarning = function deprecationWarning(deprecated, preferred) {
	  return deprecated + ' has been deprecated in favor of ' + preferred + ', please update your code';
	};
	
	// effect(type,payload) 返回{ [IO]:true, [type]:payload }
	var effect = function effect(type, payload) {
	  var _ref;
	
	  return _ref = {}, _defineProperty(_ref, IO, true), _defineProperty(_ref, type, payload), _ref;
	};
	
	// take(patternOrChannel) 返回{ [IO]:true, TAKE:{pattern} }或{ [IO]:true, TAKE:{channel} }
	// pattern为字符串、symbol类型、函数、或数组；channel为channel模块下各类channel={take,flush,close}
	// patternOrChannel接受字符串、symbol类型、函数、数组、或channel模块下各类channel={take,flush,close}
	function take() {
	  var patternOrChannel = arguments.length > 0 && arguments[0] !== undefined ? arguments[0] : '*';
	
	  if (arguments.length) {
	    (0, _utils.check)(arguments[0], _utils.is.notUndef, 
	      'take(patternOrChannel): patternOrChannel is undefined');
	  }
	  if (_utils.is.pattern(patternOrChannel)) {
	    // effect(type,payload) 返回{ [IO]:true, [type]:payload }
	    return effect(TAKE, { pattern: patternOrChannel });
	  }
	  if (_utils.is.channel(patternOrChannel)) {
	    // effect(type,payload) 返回{ [IO]:true, [type]:payload }
	    return effect(TAKE, { channel: patternOrChannel });
	  }
	  throw new Error('take(patternOrChannel): argument ' + String(patternOrChannel) 
	    + ' is not valid channel or a valid pattern');
	}
	
	// 返回{ [IO]:true, TAKE:{pattern,maybe:true} }或{ [IO]:true, TAKE:{channel,maybe:true} }
	take.maybe = function () {
	  var eff = take.apply(undefined, arguments);
	  eff[TAKE].maybe = true;
	  return eff;
	};
	
	var takem = exports.takem = (0, _utils.deprecate)(take.maybe, deprecationWarning('takem', 'take.maybe'));
	
	// put([channel],action) 返回{ [IO]:true, PUT:{channel,action} }
	function put(channel, action) {
	  if (arguments.length > 1) {
	    (0, _utils.check)(channel, _utils.is.notUndef, 'put(channel, action): argument channel is undefined');
	    (0, _utils.check)(channel, _utils.is.channel, 
	      'put(channel, action): argument ' + channel + ' is not a valid channel');
	    (0, _utils.check)(action, _utils.is.notUndef, 'put(channel, action): argument action is undefined');
	  } else {
	    (0, _utils.check)(channel, _utils.is.notUndef, 'put(action): argument action is undefined');
	    action = channel;
	    channel = null;
	  }
	
	  // effect(type,payload) 返回{ [IO]:true, [type]:payload }
	  return effect(PUT, { channel: channel, action: action });
	}
	
	// put([channel],action) 返回{ [IO]:true, PUT:{channel,action,resolve:true} }
	put.resolve = function () {
	  var eff = put.apply(undefined, arguments);
	  eff[PUT].resolve = true;
	  return eff;
	};
	
	put.sync = (0, _utils.deprecate)(put.resolve, deprecationWarning('put.sync', 'put.resolve'));
	
	// race(effects) 返回{ [IO]:true, RACE:effects }
	function race(effects) {
	  // effect(type,payload) 返回{ [IO]:true, [type]:payload }
	  return effect(RACE, effects);
	}
	
	// getFnCallDesc(meth, [context,fn] | {context,fn} | fn, args) 返回{context,fn,args}
	function getFnCallDesc(meth, fn, args) {
	  (0, _utils.check)(fn, _utils.is.notUndef, meth + ': argument fn is undefined');
	
	  var context = null;
	  if (_utils.is.array(fn)) {
	    var _fn = fn;
	
	    var _fn2 = _slicedToArray(_fn, 2);
	
	    context = _fn2[0];
	    fn = _fn2[1];
	  } else if (fn.fn) {
	    var _fn3 = fn;
	    context = _fn3.context;
	    fn = _fn3.fn;
	  }
	  (0, _utils.check)(fn, _utils.is.func, meth + ': argument ' + fn + ' is not a function');
	
	  return { context: context, fn: fn, args: args };
	}
	
	// call([context,fn] | {context,fn} | fn,...args) 返回{ [IO]:true, CALL:{context,fn,args} }
	function call(fn) {
	  for (var _len = arguments.length, args = Array(_len > 1 ? _len - 1 : 0), _key = 1; _key < _len; _key++) {
	    args[_key - 1] = arguments[_key];
	  }
	
	  // getFnCallDesc(meth, [context,fn] | {context,fn} | fn, args) 返回{context,fn,args}
	  // effect(type,payload) 返回{ [IO]:true, [type]:payload }
	  return effect(CALL, getFnCallDesc('call', fn, args));
	}
	
	// apply(context,fn,[args]) 返回{ [IO]:true, CALL:{context,fn,args} }
	function apply(context, fn) {
	  var args = arguments.length > 2 && arguments[2] !== undefined ? arguments[2] : [];
	
	  // getFnCallDesc(meth, [context,fn] | {context,fn} | fn, args) 返回{context,fn,args}
	  // effect(type,payload) 返回{ [IO]:true, [type]:payload }
	  return effect(CALL, getFnCallDesc('apply', { context: context, fn: fn }, args));
	}
	
	// cps([context,fn] | {context,fn} | fn,...args) 返回{ [IO]:true, CPS:{context,fn,args} }
	function cps(fn) {
	  for (var _len2 = arguments.length, args = Array(_len2 > 1 ? _len2 - 1 : 0), _key2 = 1; _key2 < _len2; _key2++) {
	    args[_key2 - 1] = arguments[_key2];
	  }
	
	  // getFnCallDesc(meth, [context,fn] | {context,fn} | fn, args) 返回{context,fn,args}
	  // effect(type,payload) 返回{ [IO]:true, [type]:payload }
	  return effect(CPS, getFnCallDesc('cps', fn, args));
	}
	
	// fork([context,fn] | {context,fn} | fn,...args) 返回{ [IO]:true, FORK:{context,fn,args} }
	function fork(fn) {
	  for (var _len3 = arguments.length, args = Array(_len3 > 1 ? _len3 - 1 : 0), _key3 = 1; _key3 < _len3; _key3++) {
	    args[_key3 - 1] = arguments[_key3];
	  }
	
	  // getFnCallDesc(meth, [context,fn] | {context,fn} | fn, args) 返回{context,fn,args}
	  // effect(type,payload) 返回{ [IO]:true, [type]:payload }
	  return effect(FORK, getFnCallDesc('fork', fn, args));
	}
	
	// spawn([context,fn] | {context,fn} | fn,...args) 返回{ [IO]:true, FORK:{context,fn,args,detached:true} }
	function spawn(fn) {
	  for (var _len4 = arguments.length, args = Array(_len4 > 1 ? _len4 - 1 : 0), _key4 = 1; _key4 < _len4; _key4++) {
	    args[_key4 - 1] = arguments[_key4];
	  }
	
	  // fork([context,fn] | {context,fn} | fn,...args) 返回{ [IO]:true, FORK:{context,fn,args} }
	  var eff = fork.apply(undefined, [fn].concat(args));
	  eff[FORK].detached = true;
	  return eff;
	}
	
	// join(...tasks) 若tasks为多项，返回[{ [IO]:true, JOIN:task }]；只有一项，返回{ [IO]:true, JOIN:task }
	function join() {
	  for (var _len5 = arguments.length, tasks = Array(_len5), _key5 = 0; _key5 < _len5; _key5++) {
	    tasks[_key5] = arguments[_key5];
	  }
	
	  if (tasks.length > 1) {
	    return tasks.map(function (t) {
	      return join(t);
	    });
	  }
	  var task = tasks[0];
	  (0, _utils.check)(task, _utils.is.notUndef, 'join(task): argument task is undefined');
	  (0, _utils.check)(task, _utils.is.task, 'join(task): argument ' + task 
	    + ' is not a valid Task object ' + TEST_HINT);
	
	  // effect(type,payload) 返回{ [IO]:true, [type]:payload }
	  return effect(JOIN, task);
	}
	
	// cancel(...tasks) 若tasks为多项，返回[{ [IO]:true, CANCEL:task }]；只有一项，返回{ [IO]:true, CANCEL:task }
	function cancel() {
	  for (var _len6 = arguments.length, tasks = Array(_len6), _key6 = 0; _key6 < _len6; _key6++) {
	    tasks[_key6] = arguments[_key6];
	  }
	
	  if (tasks.length > 1) {
	    return tasks.map(function (t) {
	      return cancel(t);
	    });
	  }
	  var task = tasks[0];
	  if (tasks.length === 1) {
	    (0, _utils.check)(task, _utils.is.notUndef, 'cancel(task): argument task is undefined');
	    (0, _utils.check)(task, _utils.is.task, 'cancel(task): argument ' + task 
	      + ' is not a valid Task object ' + TEST_HINT);
	  }
	
	  // effect(type,payload) 返回{ [IO]:true, [type]:payload }
	  return effect(CANCEL, task || _utils.SELF_CANCELLATION);// _utils.SELF_CANCELLATION为'@@redux-saga/SELF_CANCELLATION'
	}
	
	// select(selector,...args) 返回{ [IO]:true, SELECT:{selector,args} }
	function select(selector) {
	  for (var _len7 = arguments.length, args = Array(_len7 > 1 ? _len7 - 1 : 0), _key7 = 1; _key7 < _len7; _key7++) {
	    args[_key7 - 1] = arguments[_key7];
	  }
	
	  if (arguments.length === 0) {
	    selector = _utils.ident;// _utils.ident为function ident(v) {return v;};
	  } else {
	    (0, _utils.check)(selector, _utils.is.notUndef, 'select(selector,[...]): argument selector is undefined');
	    (0, _utils.check)(selector, _utils.is.func, 'select(selector,[...]): argument ' + selector 
	      + ' is not a function');
	  }
	
	  // effect(type,payload) 返回{ [IO]:true, [type]:payload }
	  return effect(SELECT, { selector: selector, args: args });
	}
	
	// actionChannel(pattern,[buffer]) 返回{ [IO]:true, ACTION_CHANNEL:{pattern,buffer} }
	function actionChannel(pattern, buffer) {
	  (0, _utils.check)(pattern, _utils.is.notUndef, 'actionChannel(pattern,...): argument pattern is undefined');
	  if (arguments.length > 1) {
	    (0, _utils.check)(buffer, _utils.is.notUndef, 'actionChannel(pattern, buffer): argument buffer is undefined');
	    (0, _utils.check)(buffer, _utils.is.buffer, 'actionChannel(pattern, buffer): argument ' + buffer + ' is not a valid buffer');
	  }
	
	  // effect(type,payload) 返回{ [IO]:true, [type]:payload }
	  return effect(ACTION_CHANNEL, { pattern: pattern, buffer: buffer });
	}
	
	// 返回{ [IO]:true, CANCELLED:{} }
	function cancelled() {
	  // effect(type,payload) 返回{ [IO]:true, [type]:payload }
	  return effect(CANCELLED, {});
	}
	
	// flush(channel) 返回{ [IO]:true, FLUSH:channel }
	function flush(channel) {
	  (0, _utils.check)(channel, _utils.is.channel, 'flush(channel): argument ' + channel + ' is not valid channel');
	  // effect(type,payload) 返回{ [IO]:true, [type]:payload }
	  return effect(FLUSH, channel);
	}
	
	// takeEvery(patternOrChannel,worker,...args) 
	// 返回
	// { 
	//   [IO]:true, 
	//   FORK:{context:undefined,fn:_sagaHelpers.takeEveryHelper,args:[patternOrChannel,worker,...args]} 
	// }
	function takeEvery(patternOrChannel, worker) {
	  for (var _len8 = arguments.length, args = Array(_len8 > 2 ? _len8 - 2 : 0), _key8 = 2; _key8 < _len8; _key8++) {
	    args[_key8 - 2] = arguments[_key8];
	  }
	
	  // fork([context,fn] | {context,fn} | fn,...args) 返回{ [IO]:true, FORK:{context,fn,args} }
	  return fork.apply(undefined, [_sagaHelpers.takeEveryHelper, patternOrChannel, worker].concat(args));
	}
	
	// takeEvery(patternOrChannel,worker,...args) 
	// 返回
	// { 
	//   [IO]:true, 
	//   FORK:{context:undefined,fn:_sagaHelpers.takeLatestHelper,args:[patternOrChannel,worker,...args]} 
	// }
	function takeLatest(patternOrChannel, worker) {
	  for (var _len9 = arguments.length, args = Array(_len9 > 2 ? _len9 - 2 : 0), _key9 = 2; _key9 < _len9; _key9++) {
	    args[_key9 - 2] = arguments[_key9];
	  }
	
	  // fork([context,fn] | {context,fn} | fn,...args) 返回{ [IO]:true, FORK:{context,fn,args} }
	  return fork.apply(undefined, [_sagaHelpers.takeLatestHelper, patternOrChannel, worker].concat(args));
	}
	
	// takeEvery(patternOrChannel,worker,...args) 
	// 返回
	// { 
	//   [IO]:true, 
	//   FORK:{context:undefined,fn:_sagaHelpers.throttleHelper,args:[patternOrChannel,worker,...args]} 
	// }
	function throttle(ms, pattern, worker) {
	  for (var _len10 = arguments.length, args = Array(_len10 > 3 ? _len10 - 3 : 0), _key10 = 3; _key10 < _len10; _key10++) {
	    args[_key10 - 3] = arguments[_key10];
	  }
	
	  // fork([context,fn] | {context,fn} | fn,...args) 返回{ [IO]:true, FORK:{context,fn,args} }
	  return fork.apply(undefined, [_sagaHelpers.throttleHelper, ms, pattern, worker].concat(args));
	}
	
	// 返回函数获取effect的TAKE、PUT、RACE、CALL、CPS、FORK、JOIN、CANCEL、SELECT、ACTION_CHANNEL、CANCELLED、或FLUSH属性
	var createAsEffectType = function createAsEffectType(type) {
	  return function (effect) {
	    return effect && effect[IO] && effect[type];
	  };
	};
	
	var asEffect = exports.asEffect = {
	  take: createAsEffectType(TAKE),
	  put: createAsEffectType(PUT),
	  race: createAsEffectType(RACE),
	  call: createAsEffectType(CALL),
	  cps: createAsEffectType(CPS),
	  fork: createAsEffectType(FORK),
	  join: createAsEffectType(JOIN),
	  cancel: createAsEffectType(CANCEL),
	  select: createAsEffectType(SELECT),
	  actionChannel: createAsEffectType(ACTION_CHANNEL),
	  cancelled: createAsEffectType(CANCELLED),
	  flush: createAsEffectType(FLUSH)
	};
	
### channel.js事件机制支持

#### emitter用于添加绑定函数或触发函数执行

#### channel用于存储多个绑定函数，并以特定参数执行
	
	'use strict';
	
	Object.defineProperty(exports, "__esModule", {
	  value: true
	});
	
	exports.UNDEFINED_INPUT_ERROR = exports.INVALID_BUFFER = exports.isEnd = exports.END = undefined;
	
	// 浅拷贝
	var _extends = Object.assign || function (target) { 
	  for (var i = 1; i < arguments.length; i++) { 
	    var source = arguments[i]; 
	    for (var key in source) { 
	      if (Object.prototype.hasOwnProperty.call(source, key)) { 
	        target[key] = source[key]; 
	      } 
	    } 
	  } 
	  return target; 
	};
	
	exports.emitter = emitter;
	exports.channel = channel;
	exports.eventChannel = eventChannel;
	exports.stdChannel = stdChannel;
	
	var _utils = require('./utils');
	
	var _buffers = require('./buffers');
	
	var _scheduler = require('./scheduler');
	
	var CHANNEL_END_TYPE = '@@redux-saga/CHANNEL_END';
	var END = exports.END = { type: CHANNEL_END_TYPE };
	var isEnd = exports.isEnd = function isEnd(a) {
	  return a && a.type === CHANNEL_END_TYPE;
	};
	
	// 添加订阅函数，或执行订阅函数
	function emitter() {
	  var subscribers = [];
	
	  // 向subscribers添加sub订阅函数，返回函数用于移除subscribers中添加的订阅函数
	  function subscribe(sub) {
	    subscribers.push(sub);
	    return function () {
	      return (0, _utils.remove)(subscribers, sub);
	    };
	  }
	
	  // 以参数item执行subscribers中存储的各订阅函数
	  function emit(item) {
	    var arr = subscribers.slice();
	    for (var i = 0, len = arr.length; i < len; i++) {
	      arr[i](item);
	    }
	  }
	
	  return {
	    subscribe: subscribe,
	    emit: emit
	  };
	}
	
	// 无效的buffer缓存
	var INVALID_BUFFER = exports.INVALID_BUFFER = 'invalid buffer passed to channel factory function';
	
	// 无效的action
	var UNDEFINED_INPUT_ERROR = exports.UNDEFINED_INPUT_ERROR = 'Saga was provided with an undefined action';
	if (process.env.NODE_ENV !== 'production') {
	  exports.UNDEFINED_INPUT_ERROR = UNDEFINED_INPUT_ERROR += 
	    '\nHints:\n    - check that your Action Creator returns a non-undefined value\n    ' 
	    + '- if the Saga was started using runSaga, ' 
	    + 'check that your subscribe source provides the action to its listeners\n  ';
	}
	
	// 返回{take,put,flush,close,__takers__,__closed__}
	// take(cb) buffer参数缓存有值时，执行cb函数；或将cb存入takers获取者函数缓存
	// put(action) takers获取者函数缓存有值时，以action为参数执行各获取者函数；或将action存入buffer参数缓存
	// flush(cb) buffer参数缓存有值时，执行cb函数；或执行cb({type:'@@redux-saga/CHANNEL_END'})
	// __takers__ 获取takers缓存
	// __closed__ 判断close方法是否已执行，channel频道已关闭
	function channel() {
	  // 首参缓存(通过buffers模块构建或自行构建含isEmpty、put、take方法的缓存)，或取buffers模块下固定长度的数组缓存
	  // buffer缓存用于存储takers缓存中的各获取者函数的参数，即redux机制下的action
	  var buffer = arguments.length > 0 && arguments[0] !== undefined ? 
	    arguments[0] : _buffers.buffers.fixed();
	
	  var closed = false;// 频道channel关闭标识符，close方法执行时置为真值
	  var takers = [];
	
	  // 通过isEmpty、put、take方法校验buffer缓存是否buffers模块的数据结构相同
	  (0, _utils.check)(buffer, _utils.is.buffer, INVALID_BUFFER);
	
	  function checkForbiddenStates() {
	    // close方法执行关闭频道channel后，不能再使用take方法添加taker获取者函数，也判断close是否清空takers缓存
	    if (closed && takers.length) {
	      throw (0, _utils.internalErr)('Cannot have a closed channel with pending takers');
	    }
	    // takers缓存中有获取者函数，且buffer缓存含有获取者函数的参数，报错；即takers和buffer缓存不能同时有值
	    if (takers.length && !buffer.isEmpty()) {
	      throw (0, _utils.internalErr)('Cannot have pending takers with non empty buffer');
	    }
	  }
	
	  // 若takers缓存中无获取者函数时，将input压入buffer参数列表中；input即redux机制下的action
	  // 若takers缓存中有获取者函数时，取出各获取者函数并以参数input执行，最终takers将清空
	  function put(input) {
	    checkForbiddenStates();
	    (0, _utils.check)(input, _utils.is.notUndef, UNDEFINED_INPUT_ERROR);
	    if (closed) {
	      return;
	    }
	    if (!takers.length) {
	      return buffer.put(input);
	    }
	    for (var i = 0; i < takers.length; i++) {
	      var cb = takers[i];
	      if (!cb[_utils.MATCH] || cb[_utils.MATCH](input)) {
	        takers.splice(i, 1);
	        return cb(input);
	      }
	    }
	  }
	
	  // 若buffer参数缓存中没有action时，以参数{type:'@@redux-saga/CHANNEL_END'}执行cb
	  // 若buffer参数缓存中存储有action时，取出作为cb的参数，并执行cb
	  // 若closed为否，且buffer缓存中没有action时，将cb压入takers获取者函数列表中
	  function take(cb) {
	    checkForbiddenStates();
	    (0, _utils.check)(cb, _utils.is.func, 'channel.take\'s callback must be a function');
	
	    if (closed && buffer.isEmpty()) {
	      cb(END);
	    } else if (!buffer.isEmpty()) {
	      cb(buffer.take());
	    } else {
	      takers.push(cb);
	      cb.cancel = function () {
	        return (0, _utils.remove)(takers, cb);
	      };
	    }
	  }
	
	  // 将buffer参数缓存以数组形式作为cb的参数，并执行cb函数
	  function flush(cb) {
	    checkForbiddenStates(); // TODO: check if some new state should be forbidden now
	    (0, _utils.check)(cb, _utils.is.func, 'channel.flush\' callback must be a function');
	    if (closed && buffer.isEmpty()) {
	      cb(END);
	      return;
	    }
	    cb(buffer.flush());
	  }
	
	  // 取出takers缓存中的各获取者函数，以参数{type:'@@redux-saga/CHANNEL_END'}逐个执行
	  function close() {
	    checkForbiddenStates();
	    if (!closed) {
	      closed = true;
	      if (takers.length) {
	        var arr = takers;
	        takers = [];
	        for (var i = 0, len = arr.length; i < len; i++) {
	          arr[i](END);
	        }
	      }
	    }
	  }
	
	  return { take: take, put: put, flush: flush, close: close,
	    get __takers__() {
	      return takers;
	    },
	    get __closed__() {
	      return closed;
	    }
	  };
	}
	
	// eventChannel(subscribe,[buffer],[matcher])，返回{take,flush,close}
	// 参数subscribe为let emitter = emitter();emitter.subscribe用于添加订阅函数，emitter.emit方法执行订阅函数
	// 参数buffer，可以不传，作为chan的buffer参数缓存，默认取buffers模块的none方法返回值，input不作缓存
	//    默认返回值take方法缓存taker获取者函数，flush方法无操作，emitter.emit方法间接执行takers缓存的各获取者函数
	// 参数matcher，可以不传，筛选emitter.subscribe接受的input即redux机制下的action={type:""}
	function eventChannel(subscribe) {
	  var buffer = arguments.length > 1 && arguments[1] !== undefined ? 
	    arguments[1] : _buffers.buffers.none();
	  var matcher = arguments[2];
	
	  if (arguments.length > 2) {
	    (0, _utils.check)(matcher, _utils.is.func, 'Invalid match function passed to eventChannel');
	  }
	
	  var chan = channel(buffer);
	
	  // emitter.emit方法执行订阅者函数，获取chan的takers缓存获取者函数，以input为参数，并执行
	  //    或将input存入chan的buffer参数缓存中
	  // 参数input即redux机制下的action，matcher存在用于过滤input
	  var unsubscribe = subscribe(function (input) {
	    if (isEnd(input)) {
	      chan.close();
	      return;
	    }
	    if (matcher && !matcher(input)) {
	      return;
	    }
	
	    // channel函数中takers缓存有获取者函数时，直接将input作为参数执行各获取者函数
	    // 若没有，缓存到channel函数的buffer缓存中
	    chan.put(input);
	  });
	
	  if (!_utils.is.func(unsubscribe)) {
	    throw new Error('in eventChannel: subscribe should return a function to unsubscribe');
	  }
	
	  return {
	    // take(cb) 向chan添加获取者函数taker；或者调用参数cb函数以chan中的buffer缓存为参数，并执行
	    take: chan.take,
	    // flush(cb) 调用参数cb函数以chan中的buffer缓存为参数，并执行
	    // buffer缓存无内容，以参数{type:'@@redux-saga/CHANNEL_END'}执行cb
	    flush: chan.flush,
	    // 关闭chan，并清空订阅者函数
	    close: function close() {
	      if (!chan.__closed__) {
	        chan.close();
	        unsubscribe();
	      }
	    }
	  };
	}
	
	// stdChannel(subscribe) 返回{take,flush,close}
	// take(cb,matcher)方法缓存taker获取者函数cb，并向cb添加'@@redux-saga/MATCH'属性为次参matcher
	// flush方法无操作，close方法关闭chan，并清空订阅者函数
	function stdChannel(subscribe) {
	  // chan={take,flush,close}
	  // take方法缓存taker获取者函数，flush方法无操作
	  // emitter.emit方法在订阅者函数的回调cb执行过程中，间接执行takers缓存的各获取者函数
	  var chan = eventChannel(function (cb) {
	    return subscribe(function (input) {
	      if (input[_utils.SAGA_ACTION]) {// _utils.SAGA_ACTION即'@@redux-saga/SAGA_ACTION'
	        cb(input);
	        return;
	      }
	
	      // scheduler.asap(task) 若semaphore为0，执行task任务函数及scheduler模块下queue缓存的所有任务函数
	      // 不为0，将task推入scheduler模块下queue缓存中
	      (0, _scheduler.asap)(function () {
	        return cb(input);
	      });
	    });
	  });
	
	  return _extends({}, chan, {
	    // 缓存taker获取者函数cb，并向cb添加'@@redux-saga/MATCH'属性为次参matcher
	    take: function take(cb, matcher) {
	      if (arguments.length > 1) {
	        (0, _utils.check)(matcher, _utils.is.func, 'channel.take\'s matcher argument must be a function');
	        cb[_utils.MATCH] = matcher;// _utils.MATCH即'@@redux-saga/MATCH'
	      }
	      chan.take(cb);
	    }
	  });
	}
	
### buffers.js作为缓存

	'use strict';
	
	Object.defineProperty(exports, "__esModule", {
	  value: true
	});
	exports.buffers = exports.BUFFER_OVERFLOW = undefined;
	
	var _utils = require('./utils');
	
	// ringBuffer函数，缓存arr超过limit长度限制时的错误提示
	var BUFFER_OVERFLOW = exports.BUFFER_OVERFLOW = 'Channel\'s Buffer overflow!';
	
	// ringBuffer函数，缓存arr超过limit长度限制，抛出错误
	var ON_OVERFLOW_THROW = 1;
	// ringBuffer函数，缓存arr超过limit长度限制，忽略，不作处理
	var ON_OVERFLOW_DROP = 2;
	// ringBuffer函数，缓存arr超过limit长度限制，替换处理
	var ON_OVERFLOW_SLIDE = 3;
	// ringBuffer函数，缓存arr超过limit长度限制，将arr长度扩展为2*limit
	var ON_OVERFLOW_EXPAND = 4;
	
	// 返回{ isEmpty:()=>{return true;}, put:()=>{}, take:()=>{} }
	var zeroBuffer = { isEmpty: _utils.kTrue, put: _utils.noop, take: _utils.noop };
	
	function ringBuffer() {
	  var limit = arguments.length > 0 && arguments[0] !== undefined ? arguments[0] : 10;
	  var overflowAction = arguments[1];
	
	  var arr = new Array(limit);
	  var length = 0;// 追踪arr缓存的长度
	  var pushIndex = 0;// push方法存储数组项的序号值，超过limit，自动清零
	  var popIndex = 0;// take方法获取数组项的序号值，超过limit，自动清零
	
	  // 向arr缓存的pushIndex位置添加数组项，pushIndex在不超过limit限制下加1，追踪arr缓存的长度length值加1
	  var push = function push(it) {
	    arr[pushIndex] = it;
	    pushIndex = (pushIndex + 1) % limit;
	    length++;
	  };
	
	  // 向arr缓存的popIndex位置获取数组项，popIndex在不超过limit限制下加1，追踪arr缓存的长度length值减1
	  var take = function take() {
	    if (length != 0) {
	      var it = arr[popIndex];
	      arr[popIndex] = null;
	      length--;
	      popIndex = (popIndex + 1) % limit;
	      return it;
	    }
	  };
	
	  // 从popIndex序号位置起始获取arr缓存中的所有数组项，以数组形式返回，arr数组项均置为null
	  var flush = function flush() {
	    var items = [];
	    while (length) {
	      items.push(take());
	    }
	    return items;
	  };
	
	  return {
	    // 判断arr数组缓存是否尚有数据
	    isEmpty: function isEmpty() {
	      return length == 0;
	    },
	    // arr长度小于limit，向arr添加数组项；超过，根据情况报错，或替换数组项，或数组长度翻倍
	    put: function put(it) {
	      if (length < limit) {
	        push(it);
	      } else {
	        var doubledLimit = void 0;
	        switch (overflowAction) {
	          // arr超过limit长度限制时，报错处理
	          case ON_OVERFLOW_THROW:
	            throw new Error(BUFFER_OVERFLOW);
	          // arr超过limit长度限制时，pushIndex位置数组项替换为当前压入数组的it值
	          case ON_OVERFLOW_SLIDE:
	            arr[pushIndex] = it;
	            pushIndex = (pushIndex + 1) % limit;
	            popIndex = pushIndex;
	            break;
	          // arr超过limit长度限制时，将arr长度扩展为2*limit
	          case ON_OVERFLOW_EXPAND:
	            doubledLimit = 2 * limit;
	
	            arr = flush();
	
	            length = arr.length;
	            pushIndex = arr.length;
	            popIndex = 0;
	
	            arr.length = doubledLimit;
	            limit = doubledLimit;
	
	            push(it);
	            break;
	          // 忽略，不作处理
	          default:
	          // DROP
	        }
	      }
	    },
	    take: take, 
	    flush: flush
	  };
	}
	
	var buffers = exports.buffers = {
	  // 返回{ isEmpty:()=>{return true;}, put:()=>{}, take:()=>{} } 不作缓存
	  none: function none() {
	    return zeroBuffer;
	  },
	  // 获取limit长度的数组缓存{isEmpty,put,take,flush}
	  // isEmpty判断数组缓存是否为空；put储值，当数组缓存超过limit长度时，报错；take取值；flush获取数组缓存的备份
	  fixed: function fixed(limit) {
	    return ringBuffer(limit, ON_OVERFLOW_THROW);
	  },
	  // 获取limit长度的数组缓存{isEmpty,put,take,flush}；当数组缓存超过limit长度时执行put方法，忽略，不作处理
	  dropping: function dropping(limit) {
	    return ringBuffer(limit, ON_OVERFLOW_DROP);
	  },
	  // 获取limit长度的数组缓存{isEmpty,put,take,flush}；当数组缓存超过limit长度时执行put方法，作数组项替换处理
	  sliding: function sliding(limit) {
	    return ringBuffer(limit, ON_OVERFLOW_SLIDE);
	  },
	  // 获取limit长度的数组缓存{isEmpty,put,take,flush}；当数组缓存超过limit长度时执行put方法，将数组长度翻倍
	  expanding: function expanding(initialSize) {
	    return ringBuffer(initialSize, ON_OVERFLOW_EXPAND);
	  }
	};
	
### scheduler.js管控task执行流程
	
	"use strict";
	
	Object.defineProperty(exports, "__esModule", {
	  value: true
	});
	exports.asap = asap;
	exports.suspend = suspend;
	exports.flush = flush;
	
	var queue = [];
	
	// 无task任务函数执行时，semaphore为0；task任务函数执行时，semaphore自增1；task任务函数执行完成，semaphore自减1
	var semaphore = 0;
	
	// 执行task任务，并取出queue中的所有任务函数执行，semaphore置为0
	function exec(task) {
	  try {
	    suspend();
	    task();
	  } finally {
	    flush();
	  }
	}
	
	// 若semaphore为0，执行task任务函数及queue中的所有任务函数；不为0，将task推入queue中
	function asap(task) {
	  if (!semaphore) {
	    exec(task);
	  } else {
	    queue.push(task);
	  }
	}
	
	// semaphore自增1
	function suspend() {
	  semaphore++;
	}
	
	// queue中仍有任务函数，取出并执行
	function flush() {
	  semaphore--;
	  if (!semaphore && queue.length) {
	    exec(queue.shift());
	  }
	}
	
### sagaHelpers用于创建监听特定action的saga

主要目的是控制proc.js模块mainTask主任务函数的运作流程，用于设定子任务流程的有效期

#### takeEvery(pattern,saga)

当redux派发的action匹配pattern时，以saga构建子task并执行；多次派发action，将构建多个子task并先后执行

#### takeLatest(pattern,saga)

当redux派发的action匹配pattern时，以saga构建子task并执行；
多次派发action，将取消前次构建的子task，随后新建子task并执行

#### takeLatest(delay,pattern,saga)

当redux派发的action匹配pattern时，以saga构建子task并执行；
多次派发action，延迟delay时间的action有效

	'use strict';
	
	Object.defineProperty(exports, "__esModule", {
	  value: true
	});
	exports.throttle = exports.takeLatest = exports.takeEvery = undefined;
	
	var _slicedToArray = function () { 
	  // 参数arr若为数组，获取长度为i的该数组备份；若为对象且可迭代，获取长度为i的迭代内容，构成数组返回
	  function sliceIterator(arr, i) { 
	    var _arr = []; 
	    var _n = true; 
	    var _d = false; 
	    var _e = undefined; 
	    try { 
	      for (var _i = arr[Symbol.iterator](), _s; !(_n = (_s = _i.next()).done); _n = true) { 
	        _arr.push(_s.value); 
	        if (i && _arr.length === i) break; 
	      } 
	    } catch (err) { 
	      _d = true; 
	      _e = err; 
	    } finally { 
	      try { 
	        if (!_n && _i["return"]) _i["return"](); 
	      } finally { 
	        if (_d) throw _e; 
	      } 
	    } 
	    return _arr; 
	  } 
	  return function (arr, i) { 
	    if (Array.isArray(arr)) { 
	      return arr; 
	    } else if (Symbol.iterator in Object(arr)) { 
	      return sliceIterator(arr, i); 
	    } else { 
	      throw new TypeError("Invalid attempt to destructure non-iterable instance"); 
	    } 
	  }; 
	}();
	
	exports.takeEveryHelper = takeEveryHelper;
	exports.takeLatestHelper = takeLatestHelper;
	exports.throttleHelper = throttleHelper;
	
	var _channel = require('./channel');
	
	var _utils = require('./utils');
	
	var _io = require('./io');
	
	var _buffers = require('./buffers');
	
	var done = { done: true, value: undefined };
	var qEnd = {};
	
	// fsmIterator(fsm,q0,name)
	// 参数fsm={q[i]:()=>{return [q[j],outPut,updateState]}} 设定next方法启用执行分支，
	//    分支返回值q[j]设定下一个执行分支，outPut作为当前执行分支的返回值，updateState当next方法再次执行时调用
	// 示例：next方法调用时首先执行q0，返回{return:"q1"}；再次执行next方法时，调用updateState，迭代终止
	// fsmIterator({
	//   q0: ()=>{
	//     return ["q1",{return:"q1"},updateState(arg)=>{}]
	//   },
	//   q1: ()=>{
	//     return [qEnd]
	//   },
	// }, q0)
	function fsmIterator(fsm, q0) {
	  var name = arguments.length > 2 && arguments[2] !== undefined ? arguments[2] : 'iterator';
	
	  var updateState = void 0,
	      qNext = q0;
	
	  function next(arg, error) {
	    if (qNext === qEnd) {
	      return done;
	    }
	
	    if (error) {
	      qNext = qEnd;
	      throw error;
	    } else {
	      updateState && updateState(arg);
	
	      var _fsm$qNext = fsm[qNext](),
	          _fsm$qNext2 = _slicedToArray(_fsm$qNext, 3),
	          q = _fsm$qNext2[0],
	          output = _fsm$qNext2[1],
	          _updateState = _fsm$qNext2[2];
	
	      qNext = q;
	      updateState = _updateState;
	      return qNext === qEnd ? done : output;
	    }
	  }
	
	  // utils.makeIterator(next,thro,name,isHelper)构建迭代器
	  //    返回{name,next,throw,return}；同时添加[Symbol.iterator]属性使其和原生迭代器相同，也返回{name,next,throw,return}
	  //    utils.makeIterator(next)[Symbol.iterator]()含有next方法进行迭代，throw方法抛出错误
	  //    return方法返回迭代终值，name属性迭代器名
	  return (0, _utils.makeIterator)(next, function (error) {
	    return next(null, error);
	  }, name, true);
	}
	
	function safeName(patternOrChannel) {
	  if (_utils.is.channel(patternOrChannel)) {
	    return 'channel';
	  } else if (Array.isArray(patternOrChannel)) {
	    return String(patternOrChannel.map(function (entry) {
	      return String(entry);
	    }));
	  } else {
	    return String(patternOrChannel);
	  }
	}
	
	// 构造迭代器，当派发的action满足pattern时，以worker构建子task并执行；多次派发action，将调用worker构建多个子task
	//    1 next() q1分支，返回{ done: false, value: { [IO]:true, TAKE:{ pattern | channel } } }
	//    2 next(action1) q2分支，返回{ done:false, value:{ [IO]:true, FORK:{context:undefined,fn:worker,args:[...args,action1]} } }
	//    { done:false, value:{ [IO]:true, FORK:{context:undefined,fn:worker,args:[...args,action]} } }
	//    当action为{ type: '@@redux-saga/CHANNEL_END' }时，返回{ done: true, value: undefined }
	// 以fsmIterator函数构造迭代器，takeEveryHelper调用后返回含有next方法的迭代器
	//    fsmIterator首参设定迭代器next方法的各策略分支，次参决定next方法首次调用时执行哪个策略分支
	//    各策略分支的返回值首项约定下一个执行的策略分支，第二项作为当前分支的返回值，第三项设定fsmIterator的缓存updateState
	// 首次调用next方法，执行q1分支，将fsmIterator的缓存updateState设为setAction，同时返回yTake，约定二次调用next方法时执行q2分支
	// 再次调用next方法，执行q2分支，根据next参数设置action，迭代终结或者返回yFork={context,fn:worker,args:action}
	function takeEveryHelper(patternOrChannel, worker) {
	  for (var _len = arguments.length, args = Array(_len > 2 ? _len - 2 : 0), _key = 2; _key < _len; _key++) {
	    args[_key - 2] = arguments[_key];
	  }
	
	  // io.take(patternOrChannel) 返回{ [IO]:true, TAKE:{pattern} }或{ [IO]:true, TAKE:{channel} }
	  // yTake = { done: false, value: { [IO]:true, TAKE:{ pattern | channel } } };
	  var yTake = { done: false, value: (0, _io.take)(patternOrChannel) };
	  // io.fork([context,fn] | {context,fn} | fn,...args) 返回{ [IO]:true, FORK:{context,fn,args} }
	  // yFork函数返回 { done:false, value:{ [IO]:true, FORK:{context:undefined,fn:worker,args:[...args,ac]} } }
	  var yFork = function yFork(ac) {
	    return { done: false, value: _io.fork.apply(undefined, [worker].concat(args, [ac])) };
	  };
	
	  var action = void 0,
	      setAction = function setAction(ac) {
	        return action = ac;
	      };
	
	  // 返回迭代器首次调用next方法，执行q1分支，约定q2为下次next方法执行分支，同时下次next方法执行时调用setAction处理参数
	  //    yTake作为本次next方法调用时的返回值
	  // 再次调用next方法，执行q2分支，执行setAction函数设置缓存action，action为{ type: '@@redux-saga/CHANNEL_END' }时，迭代终止
	  //    否则下次next方法重新执行q1分支，yFork(action)作为本次next方法调用时的返回值
	  return fsmIterator({
	    q1: function q1() {
	      return ['q2', yTake, setAction];
	    },
	    q2: function q2() {
	      return action === _channel.END ? [qEnd] : ['q1', yFork(action)];
	    }
	  }, 'q1', 'takeEvery(' + safeName(patternOrChannel) + ', ' + worker.name + ')');
	}
	
	// 构造迭代器，当派发的action满足pattern时，以worker构建子task并执行；多次派发action，将清除前次子task，构建新的子task并执行
	//    1 next() q1分支，返回{ done: false, value: { [IO]:true, TAKE:{ pattern | channel } } }
	//    2 next(action1) q2分支，返回{ done:false, value:{ [IO]:true, FORK:{context:undefined,fn:worker,args:[...args,action1]} } }
	//        以worker函数构建子task并返回，作为saga运行过程中next函数的参数
	//    3 next(task) q1分支，返回{ done: false, value: { [IO]:true, TAKE:{ pattern | channel } } }
	//    4 next(action2) q2分支，返回{ done: false, value: { [IO]:true, CANCEL:task } }
	//        以redux机制再次派发action，清除前次task任务
	//    5 next() q3分支，返回{ done:false, value:{ [IO]:true, FORK:{context:undefined,fn:worker,args:[...args,action2]} } }
	//    6 next(task) q1分支，返回{ done: false, value: { [IO]:true, TAKE:{ pattern | channel } } }
	//    重复步骤4-6，知道task为否值，重复步骤2
	//    或action为{ type: '@@redux-saga/CHANNEL_END' }时，迭代终止，返回{ done: true, value: undefined }
	// 以fsmIterator函数构造迭代器，takeEveryHelper调用后返回含有next方法的迭代器
	//    fsmIterator首参各分支策略，次参决定next方法首次调用时执行哪个分支策略
	//    各策略分支的返回值首项约定下一个执行的策略分支，第二项作为当前分支的返回值，第三项设定fsmIterator的缓存updateState
	// 首次调用next方法，执行q1分支，将fsmIterator的缓存updateState设为setAction
	//    同时返回yTake，约定二次调用next方法时执行q2分支
	// 再次调用next方法，执行q2分支，根据next参数设置action，将updateState设为setTask
	//    返回yFork={context,fn:worker,args:action}，约定三调用next方法时执行q1分支
	// 三次调用next方法，执行q1分支，根据next参数设置task，将updateState设为setAction
	//    同时返回yTake，约定二次调用next方法时执行q2分支
	// 四次调用next方法，执行q2分支，根据next参数设置action，返回yTake，根据前次设定的task值进入q3或q1分支
	function takeLatestHelper(patternOrChannel, worker) {
	  for (var _len2 = arguments.length, args = Array(_len2 > 2 ? _len2 - 2 : 0), _key2 = 2; _key2 < _len2; _key2++) {
	    args[_key2 - 2] = arguments[_key2];
	  }
	
	  // io.take(patternOrChannel) 返回{ [IO]:true, TAKE:{pattern} }或{ [IO]:true, TAKE:{channel} }
	  // yTake = { done: false, value: { [IO]:true, TAKE:{ pattern | channel } } };
	  var yTake = { done: false, value: (0, _io.take)(patternOrChannel) };
	  // io.fork([context,fn] | {context,fn} | fn,...args) 返回{ [IO]:true, FORK:{context,fn,args} }
	  // yFork函数返回 { done:false, value:{ [IO]:true, FORK:{context:undefined,fn:worker,args:[...args,ac]} } }
	  var yFork = function yFork(ac) {
	    return { done: false, value: _io.fork.apply(undefined, [worker].concat(args, [ac])) };
	  };
	  // io.cancel(...tasks) 若tasks为多项，返回[{ [IO]:true, CANCEL:task }]；只有一项，返回{ [IO]:true, CANCEL:task }
	  // yCancel函数返回 { done: false, value: { [IO]:true, CANCEL:task } }
	  var yCancel = function yCancel(task) {
	    return { done: false, value: (0, _io.cancel)(task) };
	  };
	
	  var task = void 0,
	      action = void 0;
	  var setTask = function setTask(t) {
	    return task = t;
	  };
	  var setAction = function setAction(ac) {
	    return action = ac;
	  };
	
	  // 返回迭代器首次调用next方法，执行q1分支，约定q2为下次next方法执行分支，同时下次next方法执行时调用setAction处理参数
	  //    yTake作为本次next方法调用时的返回值
	  // 再次调用next方法，执行q2分支，执行setAction函数设置缓存action，action为{ type: '@@redux-saga/CHANNEL_END' }时，迭代终止
	  //    task缓存有值，下次next方法执行时进入q3分支，yCancel(task)作为本次next方法调用时的返回值
	  //    否则下次next方法重新执行q1分支，同时下次next方法执行时调用setTask处理参数，yFork(action)作为本次next方法调用时的返回值
	  return fsmIterator({
	    q1: function q1() {
	      return ['q2', yTake, setAction];
	    },
	    q2: function q2() {
	      return action === _channel.END ? [qEnd] : task ? ['q3', yCancel(task)] : ['q1', yFork(action), setTask];
	    },
	    q3: function q3() {
	      return ['q1', yFork(action), setTask];
	    }
	  }, 'q1', 'takeLatest(' + safeName(patternOrChannel) + ', ' + worker.name + ')');
	}
	
	// 构造迭代器，当redux派发的action匹配pattern时，以saga构建子task并执行；多次派发action，延迟delay时间的action有效
	//    1 next() q1分支，返回{ done: false, value: { [IO]:true, ACTION_CHANNEL:{pattern,buffer} } }
	//    2 next(channel) q2分支，返回{ done: false, value: { [IO]:true, TAKE:{ pattern | channel } } }
	//    3 next(action) q3分支，返回{ done:false, value:{ [IO]:true, FORK:{context:undefined,fn:worker,args:[...args,action]} } }
	//    4 next() q4分支，返回{ done: false, value: { [IO]:true, CALL:{context:undefined,fn:util.delay,args:delayLength} } }
	//    5 next() q2分支，返回{ done: false, value: { [IO]:true, TAKE:{ pattern | channel } } }
	//    6 next(action) q3分支，返回{ done:false, value:{ [IO]:true, FORK:{context:undefined,fn:worker,args:[...args,action]} } }
	//    重复步骤4-6
	//    当q3分支action为{ type: '@@redux-saga/CHANNEL_END' }时，迭代终止，返回{ done: true, value: undefined }
	function throttleHelper(delayLength, pattern, worker) {
	  for (var _len3 = arguments.length, args = Array(_len3 > 3 ? _len3 - 3 : 0), _key3 = 3; _key3 < _len3; _key3++) {
	    args[_key3 - 3] = arguments[_key3];
	  }
	
	  var action = void 0,
	      channel = void 0;
	
	  // io.actionChannel(pattern,[buffer]) 返回{ [IO]:true, ACTION_CHANNEL:{pattern,buffer} }
	  // yActionChannel = { done: false, value: { [IO]:true, ACTION_CHANNEL:{pattern,buffer} } }
	  var yActionChannel = { done: false, value: (0, _io.actionChannel)(pattern, _buffers.buffers.sliding(1)) };
	  // io.take(patternOrChannel) 返回{ [IO]:true, TAKE:{pattern} }或{ [IO]:true, TAKE:{channel} }
	  // yTake函数返回 { done: false, value: { [IO]:true, TAKE:{ pattern | channel } } };
	  var yTake = function yTake() {
	    return { done: false, value: (0, _io.take)(channel) };
	  };
	  // io.fork([context,fn] | {context,fn} | fn,...args) 返回{ [IO]:true, FORK:{context,fn,args} }
	  // yFork函数返回 { done:false, value:{ [IO]:true, FORK:{context:undefined,fn:worker,args:[...args,ac]} } }
	  var yFork = function yFork(ac) {
	    return { done: false, value: _io.fork.apply(undefined, [worker].concat(args, [ac])) };
	  };
	  // io.call([context,fn] | {context,fn} | fn,...args) 返回{ [IO]:true, CALL:{context,fn,args} }
	  // yDelay = { done: false, value: { [IO]:true, CALL:{context:undefined,fn:util.delay,args:delayLength} } }
	  var yDelay = { done: false, value: (0, _io.call)(_utils.delay, delayLength) };
	
	  var setAction = function setAction(ac) {
	    return action = ac;
	  };
	  var setChannel = function setChannel(ch) {
	    return channel = ch;
	  };
	
	  return fsmIterator({
	    q1: function q1() {
	      return ['q2', yActionChannel, setChannel];
	    },
	    q2: function q2() {
	      return ['q3', yTake(), setAction];
	    },
	    q3: function q3() {
	      return action === _channel.END ? [qEnd] : ['q4', yFork(action)];
	    },
	    q4: function q4() {
	      return ['q2', yDelay];
	    }
	  }, 'q1', 'throttle(' + safeName(pattern) + ', ' + worker.name + ')');
	}
	
	var deprecationWarning = function deprecationWarning(helperName) {
	  return 'import ' + helperName + ' from \'redux-saga\' has been deprecated in favor of import ' 
	    + helperName + ' from \'redux-saga/effects\'.\nThe latter will not work with yield*, ' 
	    + 'as helper effects are wrapped automatically for you in fork effect.\nTherefore yield ' 
	    + helperName + ' will return task descriptor to your saga and execute next lines of code.';
	};
	var takeEvery = exports.takeEvery = (0, _utils.deprecate)(takeEveryHelper, deprecationWarning('takeEvery'));
	var takeLatest = exports.takeLatest = (0, _utils.deprecate)(takeLatestHelper, deprecationWarning('takeLatest'));
	var throttle = exports.throttle = (0, _utils.deprecate)(throttleHelper, deprecationWarning('throttle'));
	
## 对外接口

### effects.js

	'use strict';
	
	Object.defineProperty(exports, "__esModule", {
	  value: true
	});
	
	var _io = require('./internal/io');
	
	Object.defineProperty(exports, 'take', {
	  enumerable: true,
	  get: function get() {
	    return _io.take;
	  }
	});
	Object.defineProperty(exports, 'takem', {
	  enumerable: true,
	  get: function get() {
	    return _io.takem;
	  }
	});
	Object.defineProperty(exports, 'put', {
	  enumerable: true,
	  get: function get() {
	    return _io.put;
	  }
	});
	Object.defineProperty(exports, 'race', {
	  enumerable: true,
	  get: function get() {
	    return _io.race;
	  }
	});
	Object.defineProperty(exports, 'call', {
	  enumerable: true,
	  get: function get() {
	    return _io.call;
	  }
	});
	Object.defineProperty(exports, 'apply', {
	  enumerable: true,
	  get: function get() {
	    return _io.apply;
	  }
	});
	Object.defineProperty(exports, 'cps', {
	  enumerable: true,
	  get: function get() {
	    return _io.cps;
	  }
	});
	Object.defineProperty(exports, 'fork', {
	  enumerable: true,
	  get: function get() {
	    return _io.fork;
	  }
	});
	Object.defineProperty(exports, 'spawn', {
	  enumerable: true,
	  get: function get() {
	    return _io.spawn;
	  }
	});
	Object.defineProperty(exports, 'join', {
	  enumerable: true,
	  get: function get() {
	    return _io.join;
	  }
	});
	Object.defineProperty(exports, 'cancel', {
	  enumerable: true,
	  get: function get() {
	    return _io.cancel;
	  }
	});
	Object.defineProperty(exports, 'select', {
	  enumerable: true,
	  get: function get() {
	    return _io.select;
	  }
	});
	Object.defineProperty(exports, 'actionChannel', {
	  enumerable: true,
	  get: function get() {
	    return _io.actionChannel;
	  }
	});
	Object.defineProperty(exports, 'cancelled', {
	  enumerable: true,
	  get: function get() {
	    return _io.cancelled;
	  }
	});
	Object.defineProperty(exports, 'flush', {
	  enumerable: true,
	  get: function get() {
	    return _io.flush;
	  }
	});
	Object.defineProperty(exports, 'takeEvery', {
	  enumerable: true,
	  get: function get() {
	    return _io.takeEvery;
	  }
	});
	Object.defineProperty(exports, 'takeLatest', {
	  enumerable: true,
	  get: function get() {
	    return _io.takeLatest;
	  }
	});
	Object.defineProperty(exports, 'throttle', {
	  enumerable: true,
	  get: function get() {
	    return _io.throttle;
	  }
	});
	
### utils.js

	'use strict';
	
	Object.defineProperty(exports, "__esModule", {
	  value: true
	});
	
	var _utils = require('./internal/utils');
	
	Object.defineProperty(exports, 'TASK', {
	  enumerable: true,
	  get: function get() {
	    return _utils.TASK;
	  }
	});
	Object.defineProperty(exports, 'SAGA_ACTION', {
	  enumerable: true,
	  get: function get() {
	    return _utils.SAGA_ACTION;
	  }
	});
	Object.defineProperty(exports, 'noop', {
	  enumerable: true,
	  get: function get() {
	    return _utils.noop;
	  }
	});
	Object.defineProperty(exports, 'is', {
	  enumerable: true,
	  get: function get() {
	    return _utils.is;
	  }
	});
	Object.defineProperty(exports, 'deferred', {
	  enumerable: true,
	  get: function get() {
	    return _utils.deferred;
	  }
	});
	Object.defineProperty(exports, 'arrayOfDeffered', {
	  enumerable: true,
	  get: function get() {
	    return _utils.arrayOfDeffered;
	  }
	});
	Object.defineProperty(exports, 'createMockTask', {
	  enumerable: true,
	  get: function get() {
	    return _utils.createMockTask;
	  }
	});
	
	var _io = require('./internal/io');
	
	Object.defineProperty(exports, 'asEffect', {
	  enumerable: true,
	  get: function get() {
	    return _io.asEffect;
	  }
	});
	
	var _proc = require('./internal/proc');
	
	Object.defineProperty(exports, 'CHANNEL_END', {
	  enumerable: true,
	  get: function get() {
	    return _proc.CHANNEL_END;
	  }
	});
	
### index.js

	'use strict';
	
	Object.defineProperty(exports, "__esModule", {
	  value: true
	});
	exports.utils = exports.effects = exports.CANCEL = exports.delay = exports.throttle = exports.takeLatest = exports.takeEvery = exports.buffers = exports.channel = exports.eventChannel = exports.END = exports.runSaga = undefined;
	
	var _runSaga = require('./internal/runSaga');
	
	Object.defineProperty(exports, 'runSaga', {
	  enumerable: true,
	  get: function get() {
	    return _runSaga.runSaga;
	  }
	});
	
	var _channel = require('./internal/channel');
	
	Object.defineProperty(exports, 'END', {
	  enumerable: true,
	  get: function get() {
	    return _channel.END;
	  }
	});
	Object.defineProperty(exports, 'eventChannel', {
	  enumerable: true,
	  get: function get() {
	    return _channel.eventChannel;
	  }
	});
	Object.defineProperty(exports, 'channel', {
	  enumerable: true,
	  get: function get() {
	    return _channel.channel;
	  }
	});
	
	var _buffers = require('./internal/buffers');
	
	Object.defineProperty(exports, 'buffers', {
	  enumerable: true,
	  get: function get() {
	    return _buffers.buffers;
	  }
	});
	
	var _sagaHelpers = require('./internal/sagaHelpers');
	
	Object.defineProperty(exports, 'takeEvery', {
	  enumerable: true,
	  get: function get() {
	    return _sagaHelpers.takeEvery;
	  }
	});
	Object.defineProperty(exports, 'takeLatest', {
	  enumerable: true,
	  get: function get() {
	    return _sagaHelpers.takeLatest;
	  }
	});
	Object.defineProperty(exports, 'throttle', {
	  enumerable: true,
	  get: function get() {
	    return _sagaHelpers.throttle;
	  }
	});
	
	var _utils = require('./internal/utils');
	
	Object.defineProperty(exports, 'delay', {
	  enumerable: true,
	  get: function get() {
	    return _utils.delay;
	  }
	});
	Object.defineProperty(exports, 'CANCEL', {
	  enumerable: true,
	  get: function get() {
	    return _utils.CANCEL;
	  }
	});
	
	var _middleware = require('./internal/middleware');
	var _middleware2 = _interopRequireDefault(_middleware);
	
	var _effects = require('./effects');
	var effects = _interopRequireWildcard(_effects);
	
	var _utils2 = require('./utils');
	var utils = _interopRequireWildcard(_utils2);
	
	function _interopRequireWildcard(obj) { 
	  if (obj && obj.__esModule) { 
	    return obj; 
	  } else { 
	    var newObj = {}; 
	    if (obj != null) { 
	      for (var key in obj) { 
	        if (Object.prototype.hasOwnProperty.call(obj, key)) newObj[key] = obj[key]; 
	      } 
	    } 
	    newObj.default = obj; 
	    return newObj; 
	  } 
	}
	
	function _interopRequireDefault(obj) { return obj && obj.__esModule ? obj : { default: obj }; }
	
	exports.default = _middleware2.default;
	exports.effects = effects;
	exports.utils = utils;
	
## 插件引用
1. 由package.json中{"name": "redux-saga","main": "lib/index.js"} 提供require("redux-saga")功能
2. 由effects/package.json中{"name": "redux-saga/effects","main": "../lib/effects.js"} 提供require("redux-saga/effects")功能
2. 由effects/utils.json中{"name": "redux-saga/utils","main": "../lib/utils"} 提供require("redux-saga/utils")功能