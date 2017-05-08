# depd 1.1.0

## 概述及使用

depd模块调用分离的方法时调用Error.captureStackTrace提示方法名、堆栈信息。

	var deprecate = require('depd')('my-cool-module')
	
	exports.oldfunction = deprecate.function(function () {
	  // all calls to function are deprecated
	}, 'oldfunction')
	
	exports.weirdfunction = function () {
	  if (arguments.length < 2) {
	    deprecate('weirdfunction args < 2')
	  } else if (typeof arguments[0] !== 'string') {
	    deprecate('weirdfunction non-string first arg')
	  }
	}
	
	deprecate.property(exports, 'oldprop', 'oldprop >= 0.10')
	
	// 输出堆栈信息如
	<!--DeprecationError: my-cool-module deprecated oldfunction
	    at Object.<anonymous> ([eval]-wrapper:6:22)
	    at Module._compile (module.js:456:26)
	    at evalScript (node.js:532:25)
	    at startup (node.js:80:7)
	    at node.js:902:3-->
	    
## 源码

### index.js

	var callSiteToString = require('./lib/compat').callSiteToString
	var eventListenerCount = require('./lib/compat').eventListenerCount// 获取某对象绑定某时间函数的数量
	var relative = require('path').relative
	
	module.exports = depd
	
	var basePath = process.cwd()
	
	// 判断环境变量str是否含有namespace
	function containsNamespace(str, namespace) {
	  var val = str.split(/[ ,]+/)
	
	  namespace = String(namespace).toLowerCase()
	
	  for (var i = 0 ; i < val.length; i++) {
	    if (!(str = val[i])) continue;
	
	    if (str === '*' || str.toLowerCase() === namespace) {
	      return true
	    }
	  }
	
	  return false
	}
	
	// 将obj[prop]属性设置为可读可写
	function convertDataDescriptorToAccessor(obj, prop, message) {
	  var descriptor = Object.getOwnPropertyDescriptor(obj, prop)
	  var value = descriptor.value
	
	  descriptor.get = function getter() { return value }
	
	  if (descriptor.writable) {
	    descriptor.set = function setter(val) { return value = val }
	  }
	
	  delete descriptor.value
	  delete descriptor.writable
	
	  Object.defineProperty(obj, prop, descriptor)
	
	  return descriptor
	}
	
	// 返回"arg0, arg1..., arg[arity]"
	function createArgumentsString(arity) {
	  var str = ''
	
	  for (var i = 0; i < arity; i++) {
	    str += ', arg' + i
	  }
	
	  return str.substr(2)
	}
	
	// 以字符串返回被分离函数的函数名、命名空间、消息及堆栈信息
	function createStackString(stack) {
	  var str = this.name + ': ' + this.namespace
	
	  if (this.message) {
	    str += ' deprecated ' + this.message
	  }
	
	  for (var i = 0; i < stack.length; i++) {
	    str += '\n    at ' + callSiteToString(stack[i])
	  }
	
	  return str
	}
	
	function depd(namespace) {
	  if (!namespace) {
	    throw new TypeError('argument namespace is required')
	  }
	
	  var stack = getStack()// 获取错误对象的堆栈信息
	  var site = callSiteLocation(stack[1])// 返回报错时的文件名、行数、列数、函数名等信息
	  var file = site[0]
	
	  function deprecate(message) {
	    log.call(deprecate, message)
	  }
	
	  deprecate._file = file
	  deprecate._ignored = isignored(namespace)// 设为真值，忽略，不输出日志
	  deprecate._namespace = namespace
	  deprecate._traced = istraced(namespace)// 设为真值，输出堆栈信息
	  deprecate._warned = Object.create(null)// 缓存记录，调用堆栈提示过一次后，不予再次提示
	
	  deprecate.function = wrapfunction// function(fn, message) 直接提示
	  deprecate.property = wrapproperty// property(obj, prop, message) obj[prop]属性赋值取值时提示
	
	  return deprecate// 返回值调用时即打印信息，包含堆栈信息
	}
	
	// 以环境变量设定忽略报错处理的命名空间
	function isignored(namespace) {
	  if (process.noDeprecation) {
	    return true
	  }
	
	  var str = process.env.NO_DEPRECATION || ''
	
	  return containsNamespace(str, namespace)
	}
	
	// 以环境变量设定追踪堆栈信息的命名空间
	function istraced(namespace) {
	  if (process.traceDeprecation) {
	    return true
	  }
	
	  var str = process.env.TRACE_DEPRECATION || ''
	
	  return containsNamespace(str, namespace)
	}
	
	function log(message, site) {
	  // process未监听'deprecation'事件时，忽略，不输出日志
	  var haslisteners = eventListenerCount(process, 'deprecation') !== 0
	
	  if (!haslisteners && this._ignored) {
	    return
	  }
	
	  var caller
	  var callFile
	  var callSite
	  var i = 0
	  var seen = false
	  var stack = getStack()// 获取错误对象的堆栈信息
	  var file = this._file
	
	  if (site) {
	    callSite = callSiteLocation(stack[1])// 返回报错时的文件名、行数、列数、函数名等信息[file, line, colm]
	    callSite.name = site.name
	    file = callSite[0]
	  } else {
	    i = 2
	    site = callSiteLocation(stack[i])// 返回报错时的文件名、行数、列数、函数名等信息[file, line, colm]
	    callSite = site
	  }
	
	  for (; i < stack.length; i++) {
	    caller = callSiteLocation(stack[i])// 返回报错时的文件名、行数、列数、函数名等信息[file, line, colm]
	    callFile = caller[0]
	
	    if (callFile === file) {
	      seen = true
	    } else if (callFile === this._file) {
	      file = this._file
	    } else if (seen) {
	      break
	    }
	  }
	
	  var key = caller
	    ? site.join(':') + '__' + caller.join(':')
	    : undefined
	
	  if (key !== undefined && key in this._warned) {
	    return
	  }
	
	  this._warned[key] = true
	
	  if (!message) {
	    // defaultMessage(site)返回由报错堆栈stack的数组项site的文件地址、行号、列号信息转化得来的字符串
	    message = callSite === site || !callSite.name
	      ? defaultMessage(site)
	      : defaultMessage(callSite)
	  }
	
	  // 触发process的'deprecation'事件，传输错误对象
	  if (haslisteners) {
	    // DeprecationError(namespace, message, stack)
	    // 返回{constructor:DeprecationError,message,name:'DeprecationError',namespace,stack:"..."}错误对象
	    var err = DeprecationError(this._namespace, message, stack.slice(i))
	    process.emit('deprecation', err)
	    return
	  }
	
	  // format and write message
	  var format = process.stderr.isTTY
	    ? formatColor// 有彩色打印
	    : formatPlain// 无彩色打印
	  var msg = format.call(this, message, caller, stack.slice(i))
	  process.stderr.write(msg + '\n', 'utf8')
	
	  return
	}
	
	// 根据错误堆栈信息stack数组项CallSite对象，返回报错时的文件名、行数、列数、函数名等信息[file, line, colm]
	function callSiteLocation(callSite) {
	  var file = callSite.getFileName() || '<anonymous>'
	  var line = callSite.getLineNumber()
	  var colm = callSite.getColumnNumber()
	
	  if (callSite.isEval()) {
	    file = callSite.getEvalOrigin() + ', ' + file
	  }
	
	  var site = [file, line, colm]
	
	  site.callSite = callSite
	  site.name = callSite.getFunctionName()
	
	  return site
	}
	
	// 返回由报错的文件地址、行号、列号信息转化得来的字符串
	function defaultMessage(site) {
	  var callSite = site.callSite
	  var funcName = site.name
	
	  if (!funcName) {
	    // formatLocation(site)返回报错的文件地址、行号、列号信息
	    funcName = '<anonymous@' + formatLocation(site) + '>'
	  }
	
	  var context = callSite.getThis()
	  var typeName = context && callSite.getTypeName()
	
	  if (typeName === 'Object') {
	    typeName = undefined
	  }
	
	  if (typeName === 'Function') {
	    typeName = context.name || typeName
	  }
	
	  return typeName && callSite.getMethodName()
	    ? typeName + '.' + funcName
	    : funcName
	}
	
	function formatPlain(msg, caller, stack) {
	  var timestamp = new Date().toUTCString()
	
	  var formatted = timestamp
	    + ' ' + this._namespace
	    + ' deprecated ' + msg
	
	  // add stack trace
	  if (this._traced) {
	    for (var i = 0; i < stack.length; i++) {
	      formatted += '\n    at ' + callSiteToString(stack[i])
	    }
	
	    return formatted
	  }
	
	  if (caller) {
	    formatted += ' at ' + formatLocation(caller)
	  }
	
	  return formatted
	}
	
	function formatColor(msg, caller, stack) {
	  var formatted = '\x1b[36;1m' + this._namespace + '\x1b[22;39m' // bold cyan
	    + ' \x1b[33;1mdeprecated\x1b[22;39m' // bold yellow
	    + ' \x1b[0m' + msg + '\x1b[39m' // reset
	
	  // add stack trace
	  if (this._traced) {
	    for (var i = 0; i < stack.length; i++) {
	      formatted += '\n    \x1b[36mat ' + callSiteToString(stack[i]) + '\x1b[39m' // cyan
	    }
	
	    return formatted
	  }
	
	  if (caller) {
	    formatted += ' \x1b[36m' + formatLocation(caller) + '\x1b[39m' // cyan
	  }
	
	  return formatted
	}
	
	// 返回报错的文件地址、行号、列号信息
	function formatLocation(callSite) {
	  return relative(basePath, callSite[0])
	    + ':' + callSite[1]
	    + ':' + callSite[2]
	}
	
	// 获取错误对象的堆栈信息
	function getStack() {
	  var limit = Error.stackTraceLimit// 指定错误追踪的堆栈数量
	  var obj = {}
	
	  // Error.prepareStackTrace(error,stack) 传参error为错误对象，可重写
	  //    传参stack为数组形式的CallSite对象，包含错误的函数名、行数等信息，用于打印堆栈信息等
	  // Error.prepareStackTrace = function (error, structuredStackTrace) {
	  //   var trace = structuredStackTrace.map(function (callSite) {
	  //     return '    at: ' + callSite.getFunctionName() + ' ('
	  //                       + callSite.getFileName() + ':'
	  //                       + callSite.getLineNumber() + ':'
	  //                       + callSite.getColumnNumber() + ')';
	  //   });
	  //   return error.toString() + "\n" + trace.join("\n")
	  // };
	  var prep = Error.prepareStackTrace
	
	  Error.prepareStackTrace = prepareObjectStackTrace
	  Error.stackTraceLimit = Math.max(10, limit)
	
	  // Error.captureStackTrace(targetObject[, constructorOpt])
	  // 为targetObject添加stack属性，访问targetObject.stack属性时输出Error.captureStackTrace调用时的堆栈信息
	  // constructorOpt为函数，提供时，访问targetObject.stack属性时忽略constructorOpt之上及其自身的堆栈信息
	  // 示例：
	  // function MyError() {
	  //   Error.captureStackTrace(this, MyError);
	  // }
	  // new MyError().stack;// captureStackTrace若没有参数MyError，显示MyError及其上的堆栈信息；否则不显示
	  Error.captureStackTrace(obj)
	
	  // getStack除外的堆栈信息
	  var stack = obj.stack.slice(1)
	
	  Error.prepareStackTrace = prep
	  Error.stackTraceLimit = limit
	
	  return stack
	}
	
	// 返回堆栈信息，CallSite数组，可获取调用报错时的文件名、行数、列数、函数名等错误信息
	function prepareObjectStackTrace(obj, stack) {
	  return stack
	}
	
	function wrapfunction(fn, message) {
	  if (typeof fn !== 'function') {
	    throw new TypeError('argument fn must be a function')
	  }
	
	  var args = createArgumentsString(fn.length)
	  var deprecate = this
	  var stack = getStack()
	  var site = callSiteLocation(stack[1])
	
	  site.name = fn.name
	
	  var deprecatedfn = eval('(function (' + args + ') {\n'
	    + '"use strict"\n'
	    + 'log.call(deprecate, message, site)\n'
	    + 'return fn.apply(this, arguments)\n'
	    + '})')
	
	  return deprecatedfn
	}
	
	function wrapproperty(obj, prop, message) {
	  if (!obj || (typeof obj !== 'object' && typeof obj !== 'function')) {
	    throw new TypeError('argument obj must be object')
	  }
	
	  var descriptor = Object.getOwnPropertyDescriptor(obj, prop)
	
	  if (!descriptor) {
	    throw new TypeError('must call property on owner object')
	  }
	
	  if (!descriptor.configurable) {
	    throw new TypeError('property must be configurable')
	  }
	
	  var deprecate = this
	  var stack = getStack()
	  var site = callSiteLocation(stack[1])
	
	  site.name = prop
	
	  if ('value' in descriptor) {
	    descriptor = convertDataDescriptorToAccessor(obj, prop, message)
	  }
	
	  var get = descriptor.get
	  var set = descriptor.set
	
	  if (typeof get === 'function') {
	    descriptor.get = function getter() {
	      log.call(deprecate, message, site)
	      return get.apply(this, arguments)
	    }
	  }
	
	  if (typeof set === 'function') {
	    descriptor.set = function setter() {
	      log.call(deprecate, message, site)
	      return set.apply(this, arguments)
	    }
	  }
	
	  Object.defineProperty(obj, prop, descriptor)
	}
	
	// 返回{constructor:DeprecationError,message,name:'DeprecationError',namespace,stack:"..."}错误对象
	function DeprecationError(namespace, message, stack) {
	  var error = new Error()
	  var stackString
	
	  Object.defineProperty(error, 'constructor', {
	    value: DeprecationError
	  })
	
	  Object.defineProperty(error, 'message', {
	    configurable: true,
	    enumerable: false,
	    value: message,
	    writable: true
	  })
	
	  Object.defineProperty(error, 'name', {
	    enumerable: false,
	    configurable: true,
	    value: 'DeprecationError',
	    writable: true
	  })
	
	  Object.defineProperty(error, 'namespace', {
	    configurable: true,
	    enumerable: false,
	    value: namespace,
	    writable: true
	  })
	
	  Object.defineProperty(error, 'stack', {
	    configurable: true,
	    enumerable: false,
	    get: function () {
	      if (stackString !== undefined) {
	        return stackString
	      }
	
	      return stackString = createStackString.call(this, stack)
	    },
	    set: function setter(val) {
	      stackString = val
	    }
	  })
	
	  return error
	}
	
### compat/event-listener-count.js

统计绑定函数数量。

	'use strict'
	
	module.exports = eventListenerCount
	
	function eventListenerCount(emitter, type) {
	  return emitter.listeners(type).length
	}

### compat/callsite-tostring.js

获取callsite对象的文件地址、函数名、行号、列号等信息。

	'use strict'
	
	module.exports = callSiteToString
	
	function callSiteFileLocation(callSite) {
	  var fileName
	  var fileLocation = ''
	
	  if (callSite.isNative()) {
	    fileLocation = 'native'
	  } else if (callSite.isEval()) {
	    fileName = callSite.getScriptNameOrSourceURL()
	    if (!fileName) {
	      fileLocation = callSite.getEvalOrigin()
	    }
	  } else {
	    fileName = callSite.getFileName()
	  }
	
	  if (fileName) {
	    fileLocation += fileName
	
	    var lineNumber = callSite.getLineNumber()
	    if (lineNumber != null) {
	      fileLocation += ':' + lineNumber
	
	      var columnNumber = callSite.getColumnNumber()
	      if (columnNumber) {
	        fileLocation += ':' + columnNumber
	      }
	    }
	  }
	
	  return fileLocation || 'unknown source'
	}
	
	// 获取callSite对象的文件地址，函数名，方法名，行号、列号等信息
	function callSiteToString(callSite) {
	  var addSuffix = true
	  var fileLocation = callSiteFileLocation(callSite)
	  var functionName = callSite.getFunctionName()
	  var isConstructor = callSite.isConstructor()
	  var isMethodCall = !(callSite.isToplevel() || isConstructor)
	  var line = ''
	
	  if (isMethodCall) {
	    var methodName = callSite.getMethodName()
	    var typeName = getConstructorName(callSite)
	
	    if (functionName) {
	      if (typeName && functionName.indexOf(typeName) !== 0) {
	        line += typeName + '.'
	      }
	
	      line += functionName
	
	      if (methodName && functionName.lastIndexOf('.' + methodName) !== functionName.length - methodName.length - 1) {
	        line += ' [as ' + methodName + ']'
	      }
	    } else {
	      line += typeName + '.' + (methodName || '<anonymous>')
	    }
	  } else if (isConstructor) {
	    line += 'new ' + (functionName || '<anonymous>')
	  } else if (functionName) {
	    line += functionName
	  } else {
	    addSuffix = false
	    line += fileLocation
	  }
	
	  if (addSuffix) {
	    line += ' (' + fileLocation + ')'
	  }
	
	  return line
	}
	
	function getConstructorName(obj) {
	  var receiver = obj.receiver
	  return (receiver.constructor && receiver.constructor.name) || null
	}

### compat/buffer-concat.js

	'use strict'
	
	module.exports = bufferConcat
	
	function bufferConcat(bufs) {
	  var length = 0
	
	  for (var i = 0, len = bufs.length; i < len; i++) {
	    length += bufs[i].length
	  }
	
	  var buf = new Buffer(length)
	  var pos = 0
	
	  for (var i = 0, len = bufs.length; i < len; i++) {
	    bufs[i].copy(buf, pos)
	    pos += bufs[i].length
	  }
	
	  return buf
	}
	
### compat/index.js

	'use strict'
	
	var Buffer = require('buffer')
	var EventEmitter = require('events').EventEmitter
	
	lazyProperty(module.exports, 'bufferConcat', function bufferConcat() {
	  return Buffer.concat || require('./buffer-concat')
	})
	
	lazyProperty(module.exports, 'callSiteToString', function callSiteToString() {
	  var limit = Error.stackTraceLimit
	  var obj = {}
	  var prep = Error.prepareStackTrace
	
	  function prepareObjectStackTrace(obj, stack) {
	    return stack
	  }
	
	  Error.prepareStackTrace = prepareObjectStackTrace
	  Error.stackTraceLimit = 2
	
	  Error.captureStackTrace(obj)
	
	  var stack = obj.stack.slice()
	
	  Error.prepareStackTrace = prep
	  Error.stackTraceLimit = limit
	
	  return stack[0].toString ? toString : require('./callsite-tostring')
	})
	
	lazyProperty(module.exports, 'eventListenerCount', function eventListenerCount() {
	  return EventEmitter.listenerCount || require('./event-listener-count')
	})
	
	function lazyProperty(obj, prop, getter) {
	  function get() {
	    var val = getter()
	
	    Object.defineProperty(obj, prop, {
	      configurable: true,
	      enumerable: true,
	      value: val
	    })
	
	    return val
	  }
	
	  Object.defineProperty(obj, prop, {
	    configurable: true,
	    enumerable: true,
	    get: get
	  })
	}
	
	function toString(obj) {
	  return obj.toString()
	}

### broswer/index.js

浏览器端不提示。

	'use strict'
	
	module.exports = depd
	
	function depd(namespace) {
	  if (!namespace) {
	    throw new TypeError('argument namespace is required')
	  }
	
	  function deprecate(message) {
	  }
	
	  deprecate._file = undefined
	  deprecate._ignored = true
	  deprecate._namespace = namespace
	  deprecate._traced = false
	  deprecate._warned = Object.create(null)
	
	  deprecate.function = wrapfunction
	  deprecate.property = wrapproperty
	
	  return deprecate// 浏览器端无提示
	}
	
	function wrapfunction(fn, message) {
	  if (typeof fn !== 'function') {
	    throw new TypeError('argument fn must be a function')
	  }
	
	  return fn
	}
	
	function wrapproperty(obj, prop, message) {
	  if (!obj || (typeof obj !== 'object' && typeof obj !== 'function')) {
	    throw new TypeError('argument obj must be object')
	  }
	
	  var descriptor = Object.getOwnPropertyDescriptor(obj, prop)
	
	  if (!descriptor) {
	    throw new TypeError('must call property on owner object')
	  }
	
	  if (!descriptor.configurable) {
	    throw new TypeError('property must be configurable')
	  }
	
	  return
	}