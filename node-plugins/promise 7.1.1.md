# promise 7.1.1

## 概述和使用

promise模块用于处理异步逻辑。

### 构建promise实例

	var Promise = require('promise');

	var promise = new Promise(function (resolve, reject) {
	  get('http://www.google.com', function (err, res) {
	    if (err) reject(err);
	    else resolve(res);
	  });
	});

### Promise.denodeify使用 

	var fs = require('fs')

	var read = Promise.denodeify(fs.readFile)
	var write = Promise.denodeify(fs.writeFile)

	var p = read('foo.json', 'utf8')
	  .then(function (str) {
	    return write('foo.json', JSON.stringify(JSON.parse(str), null, '  '), 'utf8')
	  })

### Promise.nodeify使用，用于为不了解promise的开发者提供使用接口

	// 提供(url,callback)=>{}接口，发送ajax请求后执行callback回调
	module.exports = Promise.nodeify(awesomeAPI)
	function awesomeAPI(url, callback) {
	  var promise = new Promise(function (resolve, reject) {
		get(url, function (err, res) {
		  if (err) reject(err);
		  else resolve(res);
		});
	  });
	  promise(callbak);

	  return promise
	}

## APIS

thenable对象--带then方法的对象。
* new Promise(fn=(resolve,reject)=>{})创建promise实例，参数fn调用resolve、reject函数设定成功、失败回调的执行时机；resolve、reject均由promise模块机制传入fn中。

### 静态方法

* Promise.resolve(arg=[null|undefined|true|false|0|''|{then:()=>{}}|...])构建promise实例的工厂函数，以参数arg作为成功回调的参数；当arg为promise实例或thenable对象时，以promise模块机制等待延迟，将延迟结果作为成功回调的参数。
* Promise.reject(value)构建promise实例的工厂函数，以参数value作为失败回调的参数。
* Promise.all(arr=[{then:()=>{}}|...])构建promise实例的工厂函数，参数arr中含由promise实例或thenable对象时，等待延迟，以延迟结果构建成功、失败回调的参数数组；否则直接将arr作为成功、失败回调的参数。
* Promise.race(arr=[{then:()=>{}}|...])构建promise实例的工厂函数，参数arr中含由promise实例或thenable对象时，每个延迟结束即调用成功、失败回调；否则以数组arr长度执行多次成功、失败回调。
* Promise.denodeify(fn,num)用于生成构建promise实例的工厂函数，fn尾参以函数形式(err, res)=>{}设定成功、失败执行时机，err为真值时执行失败回调、为否值时执行成功回调；默认情况下，fn返回promise实例或thenable对象，等待延迟，以延迟结果作为成功回调参数；否则直接将fn返回值作为成功回调函数。
* Promise.nodeify(fn)返回函数(...args,[callback])=>{}，调用时将执行fn(args)，若fn返回promise对象，等待延迟，最终执行callback回调；若fn返回普通值，且callback为否值，执行错误回调；若fn返回普通纸，且callback为真值，执行callback。
* Promise.enableSynchronous()激活isPending、isFulfilled、isRejected、getValue、getReason、getState原型方法功能。
* Promise.enableSynchronous()关闭isPending、isFulfilled、isRejected、getValue、getReason、getState原型方法功能。

### 原型方法

* promise.then(onFulfilled,onRejected)设定成功、失败回调。
* promise.done(onFulfilled,onRejected)同then方法，设定成功、失败回调；遇错时报错。
* promise.finally(fn=()=>{})通过参数fn函数将内部缓存self._65设定为fn的返回值，再次调用then、done方法添加的成功、失败回调函数启用执行时，都将以fn返回值作为参数。_
* promise.catch(onRejected)设定失败回调。
* promise.nodify(callback,ctx)设定成功、失败函数为以上下文ctx执行callback。
* promise.isPending()判断异步函数是否执行中。
* promise.isFulfilled()判断异步函数是否执行成功。
* promise.isRejected()判断异步函数是否执行失败。
* promise.getValue()判断异步函数的执行结果；异步函数返回promise实例，递归取该promise实例的延迟执行结果。
* promise.getReason()判断异步函数的执行结果；异步函数返回promise实例，递归取该promise实例的延迟执行结果。
* promise.isRejected()获取异步函数执行状态，0-pending，1-resovled，2-rejected；异步函数返回promise实例，递归取该promise的延迟函数执行状态。

## 核心代码

### lib/core.js

	'use strict';

	// 同步执行多个任务
	var asap = require('asap/raw');

	function noop() {}

	var LAST_ERROR = null;// 存储最后一个错误对象，供reject函数使用，输出给onRejected回调
	var IS_ERROR = {};// getThen、tryCallOne、tryCallTwo语句执行出错时返回IS_ERROR，以调用reject回调

	// 使用try-catch语句尝试获取promise对象obj的then方法
	function getThen(obj) {
	  try {
	    return obj.then;
	  } catch (ex) {
	    LAST_ERROR = ex;
	    return IS_ERROR;
	  }
	}

	// 单参数执行fn，报错将错误对象存入LAST_ERROR，供错误回调onRejected使用
	function tryCallOne(fn, a) {
	  try {
	    return fn(a);
	  } catch (ex) {
	    LAST_ERROR = ex;
	    return IS_ERROR;
	  }
	}

	// 双参数执行fn，报错将错误对象存入LAST_ERROR，供错误回调onRejected使用
	function tryCallTwo(fn, a, b) {
	  try {
	    fn(a, b);
	  } catch (ex) {
	    LAST_ERROR = ex;
	    return IS_ERROR;
	  }
	}

	module.exports = Promise;

	// 构建promise对象，fn设定成功、失败回调的执行时机
	// 
	// 示例
	// var promise = new Promise(function (resolve, reject) {
	//   get('http://www.google.com', function (err, res) {
	//     if (err) reject(err);
	//     else resolve(res);
	//   });
	// });
	// promise.then((res)=>{
	//  console.log("response is %s",res);
	// })
	function Promise(fn) {
	  if (typeof this !== 'object') {
	    throw new TypeError('Promises must be constructed via new');
	  }
	  if (typeof fn !== 'function') {
	    throw new TypeError('not a function');
	  }

	  // 由then方法后成功、失败回调函数得来的Handler实例的存储形式，1为单个对象；2为数组
	  // 用户new出的promise对象执行then方法后，this._45变更为1；再次执行then方法，变更为2
	  this._45 = 0;

	  // 延迟函数执行状态，0-pending，1-resovled，2-rejected，3-adopted
	  // adopted状态触发情形为，延迟函数fn执行结果为promise对象
	  this._81 = 0;

	  // 延迟函数执行结果或错误对象
	  this._65 = null;

	  // 存储then方法后成功、失败回调函数得来的Handler实例
	  this._54 = null;

	  if (fn === noop) return;

	  // 为延迟函数注入resolve、reject，以设定成功、失败回调执行时机；并执行延迟函数fn
	  doResolve(fn, this);
	}
	Promise._10 = null;// 用户new出promise实例后，调用then方法时立即执行，以promise实例为参数
	Promise._97 = null;// "rejected"状态启用的钩子函数
	Promise._61 = noop;

	// 添加成功、失败回调onFulfilled、onRejected
	// 当用户new出promise的传参fn无延迟时，同步执行成功、失败回调
	// 当fn有延迟时，通过resolve、reject函数调用finale函数，异步执行成功、失败回调
	Promise.prototype.then = function(onFulfilled, onRejected) {
	  if (this.constructor !== Promise) {
	    return safeThen(this, onFulfilled, onRejected);
	  }

	  // 构建新的promise，以使用户能链式调用then方法
	  var res = new Promise(noop);

	  // new Promise(fn)时，fn执行无延迟；调用handle以处理同步逻辑，即在fn执行完成后，即启动成功、失败回调
	  handle(this, new Handler(onFulfilled, onRejected, res));
	  return res;
	};

	// then方法调用this.constructor被改变，构建新的promise作为then方法的返回值
	// 该promise通过参数函数传入resolve、reject变更初始promise的状态，并调用finale函数处理异步的成功、失败回调
	function safeThen(self, onFulfilled, onRejected) {
	  return new self.constructor(function (resolve, reject) {
	    var res = new Promise(noop);
	    res.then(resolve, reject);

	    // new Promise(fn)时，fn执行无延迟；调用handle以处理同步逻辑，即在fn执行完成后，即启动成功、失败回调
	    handle(self, new Handler(onFulfilled, onRejected, res));
	  });
	};

	// 参数：
	// 参数self为当前的promise对象(即用户new出的promise对象)
	// 参数deferred为Handler实例(用户promise执行then方法后获得)
	// 
	// 执行时机
	// then方法或safeThen函数调用时，意味用户构建promise实例的传参函数无延迟，立即调用成功、失败回调；处理同步逻辑
	// finale函数调用时，前一个promise执行完成，执行promise._54中存储的成功、失败回调；处理异步逻辑
	// 
	// 功能：
	// 将用户调用then方法添加的onFulfilled、onRejected转化为Handler实例存储于self._54中
	// 并调用handleResolved函数变更当前then方法构建的promise实例_81状态、_65成功及失败回调的传参
	// 
	// 示例：
	// promise.then方法新构建的promise对象及成功、失败回调存入p._54中
	// 通过handleResolved函数调用'asap/raw'模块，使onFulfilled1、onFulfilled2同步执行
	// let p = new Promise((resolve,reject)=>{});
	// p.then(onFulfilled1,onRejected1);
	// p.then(onFulfilled2,onRejected2);
	function handle(self, deferred) {
	  while (self._81 === 3) {
	    self = self._65;
	  }

	  // Promise._10为then方法执行时启用钩子
	  if (Promise._10) {
	    Promise._10(self);
	  }

	  // self._81=0，延迟函数执行过程中：
	  // 用户单次调用promise实例的then方法，将成功、失败函数以Handler实例形式存入self._54
	  // 用户多次调用promise实例的then方法，将Handler实例以数组形式存入self._54
	  if (self._81 === 0) {
	    if (self._45 === 0) {
	      self._45 = 1;
	      self._54 = deferred;
	      return;
	    }
	    if (self._45 === 1) {
	      self._45 = 2;
	      self._54 = [self._54, deferred];
	      return;
	    }
	    self._54.push(deferred);
	    return;
	  }

	  // 变更用户调用then方法重新构建出的promise实例的_81状态、_65成功及失败回调的传参
	  handleResolved(self, deferred);
	}

	// 参数：
	// 参数self为当前的promise对象(即用户new出的promise对象)
	// 参数deferred为Handler实例(用户promise执行then方法后获得)
	// 
	// 功能：
	// 变更用户调用then方法重新构建出的promise实例的_81状态、_65成功及失败回调的传参
	// 'asap/raw'模块用于管控反复调用then方法添加onFulfilled、onRejected函数同步执行
	// 
	// 示例：
	// 'asap/raw'模块使onFulfilled1、onFulfilled2同步执行
	// let promise = new Promise((resolve,reject)=>{});
	// promise.then(onFulfilled1,onRejected1);
	// promise.then(onFulfilled2,onRejected2);
	function handleResolved(self, deferred) {
	  // 调用'asap/raw'模块同步执行多个任务，任务由用户调用then方法转化Handler实例得来
	  asap(function() {
	    // 执行状态设为resolved，用户设定的延迟函数执行完成，调用then方法的参数onFulfilled；否则调用onRejected
	    var cb = self._81 === 1 ? deferred.onFulfilled : deferred.onRejected;

	    // 链式调用then方法时，将重新构建promise对象并作为用户端的返回值
	    // 调用resolve(deferred.promise,self._65)将then方法构建出的promise状态变更为resolved，同时传入延迟函数执行结果
	    if (cb === null) {
	      if (self._81 === 1) {
	        resolve(deferred.promise, self._65);
	      } else {
	        reject(deferred.promise, self._65);
	      }
	      return;
	    }

	    // 以self._65即延迟函数执行结果作为onFulfilled、onRejected的参数，并执行onFulfilled、onRejected
	    var ret = tryCallOne(cb, self._65);

	    // 变更then方法重新构建的promise实例的_81状态、_65成功及失败回调的传参
	    if (ret === IS_ERROR) {
	      reject(deferred.promise, LAST_ERROR);
	    } else {
	      resolve(deferred.promise, ret);
	    }
	  });
	}

	// 延迟函数返回值为promise对象时，通过self._65存取由handle函数等待下一次延迟执行完成
	// 延迟函数返回值为带then方法的对象时，以该then方法设定下一次延迟的执行机制
	//    并改变用户最先new出promise实例self的状态，执行onFulfilled、onRejected回调
	// 延迟函数返回值为普通值时，改变用户最先new出promise实例self的状态，执行onFulfilled、onRejected回调
	function resolve(self, newValue) {
	  if (newValue === self) {
	    return reject(
	      self,
	      new TypeError('A promise cannot be resolved with itself.')
	    );
	  }

	  if ( newValue && (typeof newValue === 'object' || typeof newValue === 'function') ) {
	    var then = getThen(newValue);
	    if (then === IS_ERROR) {
	      return reject(self, LAST_ERROR);
	    }

	    // 当延迟函数fn返回promise实例时，handle函数迭代取值self._65获得返回的promise实例，以执行该promise中的延迟函数
	    if ( then === self.then && newValue instanceof Promise ) {
	      self._81 = 3;
	      self._65 = newValue;
	      finale(self);
	      return;

	    // 延迟函数fn返回值不是promise实例，且then方法为函数；通过doResolve设定下一次异步执行逻辑
	    // 下一次延迟函数执行过程将改变用户最先new出promise实例self的状态，通过该状态启用成功或失败回调
	    } else if (typeof then === 'function') {
	      doResolve(then.bind(newValue), self);
	      return;
	    }
	  }
	  self._81 = 1;
	  self._65 = newValue;

	  // 执行then方法添加的成功、失败回调函数
	  finale(self);
	}

	// 将self._81设为"rejected"，self._65设为延迟函数返回值或错误对象
	// 执行Promise._97钩子函数，即监听"rejected"状态的绑定函数
	function reject(self, newValue) {
	  self._81 = 2;// 执行状态设为rejected
	  self._65 = newValue;// 延迟函数返回值或错误对象

	  // Promise._97为rejected状态启用钩子
	  if (Promise._97) {
	    Promise._97(self, newValue);
	  }
	  
	  finale(self);
	}

	// 用户new出promise时，传参fn有延迟时，通过finale执行then方法添加的onFulfilled、onRejected回调
	function finale(self) {
	  if (self._45 === 1) {
	    handle(self, self._54);
	    self._54 = null;
	  }
	  if (self._45 === 2) {
	    for (var i = 0; i < self._54.length; i++) {
	      handle(self, self._54[i]);
	    }
	    self._54 = null;
	  }
	}

	// 构建Handler实例存储promise.then(onFulfilled,onRejected)注入或创建的onFulfilled、onRejected、promise
	function Handler(onFulfilled, onRejected, promise){
	  this.onFulfilled = typeof onFulfilled === 'function' ? onFulfilled : null;
	  this.onRejected = typeof onRejected === 'function' ? onRejected : null;
	  this.promise = promise;
	}

	// 向new Promise(fn)的参数fn中传入回调函数，以设定"resolved"、“rejected”状态变更时机；同时执行fn，即延迟函数
	function doResolve(fn, promise) {
	  // done标识符用于避免用户多次执行fn的回调，以及fn执行出错时，阻止异步回调改变"resolved"、“rejected”状态
	  var done = false;

	  // 向fn传入参数回调函数，用于设定延迟函数fn执行过程中何时变更为"resolved"、“rejected”状态
	  var res = tryCallTwo(fn, function (value) {
	    if (done) return;
	    done = true;
	    resolve(promise, value);
	  }, function (reason) {
	    if (done) return;
	    done = true;
	    reject(promise, reason);
	  })

	  // 调用tryCatchTwo语句执行fn函数时报错(包括fn函数执行过程中报错)，立即执行内部回调reject函数
	  if (!done && res === IS_ERROR) {
	    done = true;
	    reject(promise, LAST_ERROR);
	  }
	}

### lib/done.js

	'use strict';

	var Promise = require('./core.js');

	module.exports = Promise;

	// 同then方法，添加成功、失败回调函数；不同于then方法的时，done失败时将报错
	Promise.prototype.done = function (onFulfilled, onRejected) {
	  var self = arguments.length ? this.then.apply(this, arguments) : this;
	  self.then(null, function (err) {
	    setTimeout(function () {
	      throw err;
	    }, 0);
	  });
	};

### lib/finally.js 

	'use strict';

	var Promise = require('./core.js');

	module.exports = Promise;

	// 以函数f返回值作为下次then添加的成功、失败回调的参数；执行过程存在语法错误时，报错
	// 
	// let p = new Promise((resolve,reject)=>{});
	// p.finally(()=>{return "hello world"})
	// 	.then((res)=>{console.log(res)}) 打印"hello world"
	Promise.prototype['finally'] = function (f) {
	  return this.then(function (value) {
	  	// 由promise机制等待Promise.resolve().then()返回promise延迟执行完成
	    return Promise.resolve(f()).then(function () {
	      return value;
	    });
	  }, function (err) {
	    return Promise.resolve(f()).then(function () {
	      throw err;
	    });
	  });
	};


### lib/es6-extensions.js 

	'use strict';

	var Promise = require('./core.js');

	module.exports = Promise;

	var TRUE = valuePromise(true);
	var FALSE = valuePromise(false);
	var NULL = valuePromise(null);
	var UNDEFINED = valuePromise(undefined);
	var ZERO = valuePromise(0);
	var EMPTYSTRING = valuePromise('');

	// 伪promise实例，只能接受成功回调，失败回调无效；且成功回调执行参数设定为value
	function valuePromise(value) {
	  var p = new Promise(Promise._61);
	  p._81 = 1;
	  p._65 = value;
	  return p;
	}

	// value为对象或函数时，且含有then方法；以then方法设定成功、失败回调执行时机；构建promise实例返回
	// 其他，构建伪promise返回，只接受成功回调，且成功回调执行参数设定为value
	Promise.resolve = function (value) {
	  if (value instanceof Promise) return value;

	  // 构建伪promise实例，只能接受成功回调，且成功回调执行参数设定为value；失败回调无效
	  if (value === null) return NULL;
	  if (value === undefined) return UNDEFINED;
	  if (value === true) return TRUE;
	  if (value === false) return FALSE;
	  if (value === 0) return ZERO;
	  if (value === '') return EMPTYSTRING;

	  // value为对象或函数时，且含有then方法；以then方法设定成功、失败回调执行时机；构建promise实例返回
	  if (typeof value === 'object' || typeof value === 'function') {
	    try {
	      var then = value.then;
	      if (typeof then === 'function') {
	        return new Promise(then.bind(value));
	      }
	    } catch (ex) {
	      return new Promise(function (resolve, reject) {
	        reject(ex);
	      });
	    }
	  }
	  return valuePromise(value);
	};

	// 工厂函数，同步执行多个延迟函数、以延迟函数返回值或纯以arr数组作为成功、失败回调的参数
	// arr中含有promise实例或带then方法的延迟函数，等待延迟执行完成，以返回值作为成功、失败回调的参数
	// arr数组春伟普通值，以arr作为成功、失败回调的参数
	Promise.all = function (arr) {
	  var args = Array.prototype.slice.call(arr);

	  return new Promise(function (resolve, reject) {
	    if (args.length === 0) return resolve([]);
	    var remaining = args.length;
	    function res(i, val) {
	      if (val && (typeof val === 'object' || typeof val === 'function')) {
	        if (val instanceof Promise && val.then === Promise.prototype.then) {
	          while (val._81 === 3) {
	            val = val._65;
	          }
	          if (val._81 === 1) return res(i, val._65);
	          if (val._81 === 2) reject(val._65);
	          val.then(function (val) {
	            res(i, val);
	          }, reject);
	          return;
	        } else {
	          var then = val.then;
	          if (typeof then === 'function') {
	            var p = new Promise(then.bind(val));
	            p.then(function (val) {
	              res(i, val);
	            }, reject);
	            return;
	          }
	        }
	      }
	      args[i] = val;
	      if (--remaining === 0) {
	        resolve(args);
	      }
	    }
	    for (var i = 0; i < args.length; i++) {
	      res(i, args[i]);
	    }
	  });
	};

	// 工厂函数，构建promise实例，只能设置失败回调，且失败回调的参数固定为value；成功回调无效
	// 
	// 示例
	// let p = Promise.reject(err);
	// p.then(null,(err)=>{throw err;});
	Promise.reject = function (value) {
	  return new Promise(function (resolve, reject) {
	    reject(value);
	  });
	};

	// 工厂函数，构建promise实例，values含有多个promise对象时，各延迟执行完成调用成功、失败回调
	// values为普通值，按values数组项数目执行多次成功、失败回调
	Promise.race = function (values) {
	  return new Promise(function (resolve, reject) {
	    values.forEach(function(value){
	      Promise.resolve(value).then(resolve, reject);
	    });
	  });
	};

	// catch实例方法，添加失败时执行的回调函数
	Promise.prototype['catch'] = function (onRejected) {
	  return this.then(null, onRejected);
	};

### lib/node-extensions.js 

	'use strict';

	// This file contains then/promise specific extensions that are only useful
	// for node.js interop

	var Promise = require('./core.js');
	var asap = require('asap');

	module.exports = Promise;

	// 构建promise的工厂函数并返回，工厂函数中的参数将顺序传入fn中，以根据fn返回值有无延迟、构成成功失败回调的传参
	// fn函数尾参callbackFn，可用于设定成功、失败回调的执行时机
	// 默认fn返回promise实例或带then方法的对象时，异步处理；否则同步
	Promise.denodeify = function (fn, argumentCount) {
	  if ( typeof argumentCount === 'number' && argumentCount !== Infinity ) {
	    return denodeifyWithCount(fn, argumentCount);
	  } else {
	    return denodeifyWithoutCount(fn);
	  }
	}

	var callbackFn = (
	  'function (err, res) {' +
	  'if (err) { rj(err); } else { rs(res); }' +
	  '}'
	);

	// 返回创建promise的工厂函数，接受argumentCount数目参数，传入fn中
	// fn返回值promise实例、带then方法的对象、或普通值，等待延迟函数的返回值或直接将返回值作为成功回调的参数
	// 同时fn尾参callbackFn可在fn函数内部设定成功、失败回调的执行时机
	function denodeifyWithCount(fn, argumentCount) {
	  var args = [];
	  for (var i = 0; i < argumentCount; i++) {
	    args.push('a' + i);
	  }
	  var body = [
	    'return function (' + args.join(',') + ') {',
	    'var self = this;',
	    'return new Promise(function (rs, rj) {',
	    'var res = fn.call(',
	    ['self'].concat(args).concat([callbackFn]).join(','),
	    ');',
	    'if (res &&',
	    '(typeof res === "object" || typeof res === "function") &&',
	    'typeof res.then === "function"',
	    ') {rs(res);}',
	    '});',
	    '};'
	  ].join('');

	  // 返回(args)=>{}，用于构建promise实例，args顺序传入fn中，fn返回值作为成功回调的参数
	  // 当fn返回值为promise实例或含有then方法的延迟函数，等待延迟函数结果，再执行成功回调
	  // 同时fn尾参callbackFn可在fn函数内部设定成功、失败回调的执行时机
	  // 
	  // 拼接字符串形式设定函数体，为处理参数使用
	  return Function(['Promise', 'fn'], body)(Promise, fn);
	}

	// 构建工厂函数，不同于denodeifyWithCount的是，该工厂函数可接受不定数目的传参
	function denodeifyWithoutCount(fn) {
	  var fnLength = Math.max(fn.length - 1, 3);
	  var args = [];
	  for (var i = 0; i < fnLength; i++) {
	    args.push('a' + i);
	  }
	  var body = [
	    'return function (' + args.join(',') + ') {',
	    'var self = this;',
	    'var args;',
	    'var argLength = arguments.length;',
	    'if (arguments.length > ' + fnLength + ') {',
	    'args = new Array(arguments.length + 1);',
	    'for (var i = 0; i < arguments.length; i++) {',
	    'args[i] = arguments[i];',
	    '}',
	    '}',
	    'return new Promise(function (rs, rj) {',
	    'var cb = ' + callbackFn + ';',
	    'var res;',
	    'switch (argLength) {',
	    args.concat(['extra']).map(function (_, index) {
	      return (
	        'case ' + (index) + ':' +
	        'res = fn.call(' + ['self'].concat(args.slice(0, index)).concat('cb').join(',') + ');' +
	        'break;'
	      );
	    }).join(''),
	    'default:',
	    'args[argLength] = cb;',
	    'res = fn.apply(self, args);',
	    '}',
	    
	    'if (res &&',
	    '(typeof res === "object" || typeof res === "function") &&',
	    'typeof res.then === "function"',
	    ') {rs(res);}',
	    '});',
	    '};'
	  ].join('');

	  // 拼接字符串形式设定函数体，为处理参数、及swtich语句使用
	  return Function(['Promise', 'fn'],body)(Promise, fn);
	}

	// fn返回promise实例，支持异步逻辑，通过nodeify方法执行callback.call(ctx,null,value)
	// fn返回不是promise实例，仅可同步逻辑
	//    callback为否值，构建promise实例执行失败回调
	//    callback为真值，执行callback
	Promise.nodeify = function (fn) {
	  return function () {
	    var args = Array.prototype.slice.call(arguments);
	    var callback =
	      typeof args[args.length - 1] === 'function' ? args.pop() : null;
	    var ctx = this;
	    try {
	      return fn.apply(this, arguments).nodeify(callback, ctx);
	    } catch (ex) {
	      if (callback === null || typeof callback == 'undefined') {
	        return new Promise(function (resolve, reject) {
	          reject(ex);
	        });
	      } else {
	        asap(function () {
	          callback.call(ctx, ex);
	        })
	      }
	    }
	  }
	}

	// 多次调用nodeify方法时，成功或失败时，将同步执行多个callback函数
	// 示例：成功时将同步执行callback1.call(ctx1,null,value)、callback2.call(ctx2,null,value)
	// let promise = new Promise((resolve,reject)=>{});
	// promise.nodeify(callback1,ctx1);
	// promise.nodeify(callback2,ctx2);
	Promise.prototype.nodeify = function (callback, ctx) {
	  if (typeof callback != 'function') return this;

	  this.then(function (value) {
	    asap(function () {
	      callback.call(ctx, null, value);
	    });
	  }, function (err) {
	    asap(function () {
	      callback.call(ctx, err);
	    });
	  });
	}

### lib/synchronous.js 

	'use strict';

	var Promise = require('./core.js');

	module.exports = Promise;

	// 静态方法enableSynchronous使获取状态、获取延迟函数执行结果的各方法可用
	Promise.enableSynchronous = function () {
	  // 是否执行中
	  Promise.prototype.isPending = function() {
	    return this.getState() == 0;
	  };

	  // 是否成功
	  Promise.prototype.isFulfilled = function() {
	    return this.getState() == 1;
	  };

	  // 是否失败
	  Promise.prototype.isRejected = function() {
	    return this.getState() == 2;
	  };

	  // 获取延迟函数返回值
	  Promise.prototype.getValue = function () {
	    // 延迟函数返回promise，获取该promise延迟的返回值
	    if (this._81 === 3) {
	      return this._65.getValue();
	    }

	    if (!this.isFulfilled()) {
	      throw new Error('Cannot get a value of an unfulfilled promise.');
	    }

	    return this._65;
	  };
	  
	  // 获取延迟函数返回值
	  Promise.prototype.getReason = function () {
	    if (this._81 === 3) {
	      return this._65.getReason();
	    }

	    if (!this.isRejected()) {
	      throw new Error('Cannot get a rejection reason of a non-rejected promise.');
	    }

	    return this._65;
	  };

	  // 延迟函数执行状态，0-pending，1-resovled，2-rejected
	  Promise.prototype.getState = function () {
	    // 返回promise对象，获取该promise对象的状态
	    if (this._81 === 3) {
	      return this._65.getState();
	    }
	    if (this._81 === -1 || this._81 === -2) {
	      return 0;
	    }

	    return this._81;
	  };
	};

	// 静态方法enableSynchronous使获取状态、获取延迟函数执行结果的各方法不可用
	Promise.disableSynchronous = function() {
	  Promise.prototype.isPending = undefined;
	  Promise.prototype.isFulfilled = undefined;
	  Promise.prototype.isRejected = undefined;
	  Promise.prototype.getValue = undefined;
	  Promise.prototype.getReason = undefined;
	  Promise.prototype.getState = undefined;
	};

### lib/index.js 

	'use strict';

	module.exports = require('./core.js');// Promise构造函数、实例promise添加then方法
	require('./done.js');// 实例promise添加done方法
	require('./finally.js');// 实例promise添加finally方法
	require('./es6-extensions.js');// Promise添加静态方法，即工厂函数resolve、all、reject、race，原型方法catch
	require('./node-extensions.js');// Promise添加静态方法，即工厂函数denodeify、非工厂函数nodeify，原型方法nodeify
	require('./synchronous.js');// Promise添加静态方法enableSynchronous、disableSynchronous

## 输出接口

### index.js 

	'use strict';

	module.exports = require('./lib')

### core.js 

	'use strict';

	module.exports = require('./lib/core.js');

	console.error('require("promise/core") is deprecated, use require("promise/lib/core") instead.');

### ployfill.js 浏览器端使用

	var asap = require('asap');

	if (typeof Promise === 'undefined') {
	  Promise = require('./lib/core.js')
	  require('./lib/es6-extensions.js')
	}

	require('./polyfill-done.js');

### polyfill-done.js 

	if (typeof Promise.prototype.done !== 'function') {
	  Promise.prototype.done = function (onFulfilled, onRejected) {
	    var self = arguments.length ? this.then.apply(this, arguments) : this
	    self.then(null, function (err) {
	      setTimeout(function () {
	        throw err
	      }, 0)
	    })
	  }
	}