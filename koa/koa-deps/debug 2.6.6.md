# debug 2.6.6

## 概述和使用

debug插件用于打印日志，携带命名空间(即模块名)，前后debug的时间差。在浏览器端，根据浏览器环境判断是否开启带颜色输出，localStorage存取待调试的模块；在node端，通过环境变量设置是否开启带颜色输出，环境变量存取待调试的模块。

### 使用环境变量DEBUG=koa设置待调试模块

	var debug = require("debug")("koa");
	debug('use %s', name);// 以参数name替换%s，输出"koa use " + name + "1970-01-01 00:00:00";
	
### 使用enable方法设置待调试模块

	require("debug").enable("koa")；
	var debug = require("debug")("koa");
	debug('use %s', name);// 以参数name替换%s，输出"koa use " + name + "1970-01-01 00:00:00";
	
## 核心源码

### debug.js

浏览器和node端的基础依赖，用于构建debug函数；enable方法添加待调试的模块。

	exports = module.exports = createDebug.debug = createDebug['default'] = createDebug;
	exports.coerce = coerce;
	exports.disable = disable;
	exports.enable = enable;
	exports.enabled = enabled;
	exports.humanize = require('ms');
	
	exports.names = [];// 存储待debug的模块名，即命名空间
	exports.skips = [];// 存储无需debug的模块名，即命名空间
	
	// 存储debug函数首参字符串中如%o、%O、%j需要做特殊处理的函数；node、browser模块针对各环境做赋值
	exports.formatters = {};
	
	var prevTime;
	
	// 用于设置颜色this.color
	function selectColor(namespace) {
	  var hash = 0, i;
	
	  for (i in namespace) {
	    hash  = ((hash << 5) - hash) + namespace.charCodeAt(i);
	    hash |= 0; // Convert to 32bit integer
	  }
	
	  return exports.colors[Math.abs(hash) % exports.colors.length];
	}
	
	// 构建debug函数并作为返回值，调用时打印日志
	function createDebug(namespace) {
	
	  // debug(msg,...args)或debug(err,...args)使用args替换msg文案或err.stack或err.message中的%[a-zA-Z]
	  // exports.formatters设置特殊替换规则，如%j浏览器端将json对象字符串化等
	  function debug() {
	    // disabled?
	    if (!debug.enabled) return;
	
	    var self = debug;
	
	    var curr = +new Date();
	    var ms = curr - (prevTime || curr);
	    self.diff = ms;// 前后两次debug打印日志的时间差
	    self.prev = prevTime;// 前次debug打印日志的时间
	    self.curr = curr;// 当前debug打印日志的时间
	    prevTime = curr;
	
	    // turn the `arguments` into a proper Array
	    var args = new Array(arguments.length);
	    for (var i = 0; i < args.length; i++) {
	      args[i] = arguments[i];
	    }
	
	    // 转化首参；首参若为错误对象，返回stack或message属性；若不是错误对象，原样输出
	    args[0] = exports.coerce(args[0]);
	
	    if ('string' !== typeof args[0]) {
	      args.unshift('%O');
	    }
	
	    // 替换首参字符串中的%[a-zA-Z]；%o、%O、%j对参数args[index]作特殊处理
	    var index = 0;
	    args[0] = args[0].replace(/%([a-zA-Z%])/g, function(match, format) {
	      if (match === '%%') return match;
	      index++;
	
	      // formatter函数用于转化debug(msg,...args)函数中的参数args[index]，用以替换msg中的%[a-zA-Z]
	      var formatter = exports.formatters[format];
	      if ('function' === typeof formatter) {
	        var val = args[index];
	        match = formatter.call(self, val);
	
	        args.splice(index, 1);
	        index--;
	      }
	      return match;
	    });
	
	    // apply env-specific formatting (colors, etc.)
	    exports.formatArgs.call(self, args);
	
	    // 控制台打印日志，优先级依次为debug.log、exports.log、console.log
	    // exports.log在浏览器环境即console.log，node端为stream.write
	    var logFn = debug.log || exports.log || console.log.bind(console);
	    logFn.apply(self, args);
	  }
	
	  debug.namespace = namespace;// 命名空间
	  debug.enabled = exports.enabled(namespace);// 已使用enable(namespace)将namespace添加到exports.names为真值
	  debug.useColors = exports.useColors();// 打印日志是否带颜色
	  debug.color = selectColor(namespace);// 带颜色打印日志，debug.color色彩打印命名空间和时间错
	
	  // env-specific initialization logic for debug instances
	  if ('function' === typeof exports.init) {
	    exports.init(debug);
	  }
	
	  return debug;
	}
	
	// enable(namespaces)以","分割字符串存入exports.names|skips中，exports.skips中存储的namespaces以"-"起始
	// require("debug")(namespace)后未开启打印日志功能，需提前调用require("debug").enable(namespace)方开启
	function enable(namespaces) {
	  // exports.save(namespaces)，node端将process.env.DEBUG赋值为namespaces
	  // 浏览器端向window.localStorage存储zhong注入{debug:namespaces}
	  exports.save(namespaces);
	
	  exports.names = [];
	  exports.skips = [];
	
	  var split = (typeof namespaces === 'string' ? namespaces : '').split(/[\s,]+/);
	  var len = split.length;
	
	  for (var i = 0; i < len; i++) {
	    if (!split[i]) continue; // ignore empty strings
	    namespaces = split[i].replace(/\*/g, '.*?');
	    if (namespaces[0] === '-') {
	      exports.skips.push(new RegExp('^' + namespaces.substr(1) + '$'));
	    } else {
	      exports.names.push(new RegExp('^' + namespaces + '$'));
	    }
	  }
	}
	
	// disable()，因exports.names|skips均不接受""，该方法始终返回否值
	function disable() {
	  exports.enable('');
	}
	
	// enabled(name)，若模块名name在exports.names中，返回true；若name在exports.skips中，返回false
	function enabled(name) {
	  var i, len;
	  for (i = 0, len = exports.skips.length; i < len; i++) {
	    if (exports.skips[i].test(name)) {
	      return false;
	    }
	  }
	  for (i = 0, len = exports.names.length; i < len; i++) {
	    if (exports.names[i].test(name)) {
	      return true;
	    }
	  }
	  return false;
	}
	
	// coerce(val)，val若为错误对象，返回错误对象的stack堆栈信息或message信息；否则将val原样输出
	function coerce(val) {
	  if (val instanceof Error) return val.stack || val.message;
	  return val;
	}

### node.js

node端根据环境变量判断是否启用颜色、根据环境变量决定打印日志的方法(使用标准输出流、错误流还是将日志打印到文件中)、参数转换处理(%o|%O带颜色输出、%j序列化json对象、%d数值替换、%s字符串替换)等。
	
	var tty = require('tty');
	var util = require('util');
	
	exports = module.exports = require('./debug');
	exports.init = init;
	exports.log = log;
	exports.formatArgs = formatArgs;
	exports.save = save;
	exports.load = load;
	exports.useColors = useColors;
	
	// 可选择的颜色
	exports.colors = [6, 2, 3, 4, 5, 1];
	
	// 将环境变量转化为对象形式
	// 如DEBUG_COLORS=no node script.js将转化成{color:false}，作为输出日志的颜色配置
	exports.inspectOpts = Object.keys(process.env).filter(function (key) {
	  return /^debug_/i.test(key);
	}).reduce(function (obj, key) {
	  var prop = key
	    .substring(6)
	    .toLowerCase()
	    .replace(/_([a-z])/g, function (_, k) { return k.toUpperCase() });
	
	  var val = process.env[key];
	  if (/^(yes|on|true|enabled)$/i.test(val)) val = true;
	  else if (/^(no|off|false|disabled)$/i.test(val)) val = false;
	  else if (val === 'null') val = null;
	  else val = Number(val);
	
	  obj[prop] = val;
	  return obj;
	}, {});
	
	// DEBUG_FD=3 node script.js 3>debug.log执行时读取环境变量DEBUG_FD，文件描述符，判断文件是否和终端关联
	var fd = parseInt(process.env.DEBUG_FD, 10) || 2;
	
	if (1 !== fd && 2 !== fd) {
	  // util.deprecate(fn,msg)装饰已废弃的方法fn，调用时提示msg
	  util.deprecate(function(){}, 'except for stderr(2) and stdout(1), any other usage of DEBUG_FD is deprecated. Override debug.log if you want to use a different log function (https://git.io/debug_fd)')()
	}
	
	// 由环境变量process.env.DEBUG_FD设置打印日志的方式
	var stream = 1 === fd ? process.stdout :// 标准输出
	             2 === fd ? process.stderr :// 标准错误输出
	             createWritableStdioStream(fd);// 将日志输出到文件描述符为fd的文件中
	
	// 通过环境变量设置是否开启带颜色输出，DEBUG_COLORS=true设置或DEBUG_FD=3，且文件描述符3指向的文件被系统读取状态
	function useColors() {
	  return 'colors' in exports.inspectOpts
	    ? Boolean(exports.inspectOpts.colors)
	    : tty.isatty(fd);// 文件描述符fd，维护着进程打开文件的索引；若文件和终端关联，tty.isatty(fd)返回true
	}
	
	// debug(msg,...args)若msg中含有%O，且this.useColors为true，带ANSI颜色替换参数；控制在一行内
	exports.formatters.o = function(v) {
	  this.inspectOpts.colors = this.useColors;
	  return util.inspect(v, this.inspectOpts)
	    .replace(/\s*\n\s*/g, ' ');
	};
	
	// debug(msg,...args)若msg中含有%O，且this.useColors为true，带ANSI颜色替换参数
	exports.formatters.O = function(v) {
	  this.inspectOpts.colors = this.useColors;
	  return util.inspect(v, this.inspectOpts);
	};
	
	
	// 日志文案前插入命名空间，后添加时间戳，命名空间和时间戳均带颜色this.color输出，用户指定的日志文案默认颜色输出
	// node端以console.log('  \u001b[31;1m' + str  + '\u001b[0m')方式打印带颜色文案
	function formatArgs(args) {
	  var name = this.namespace;
	  var useColors = this.useColors;
	
	  if (useColors) {
	    var c = this.color;
	    var prefix = '  \u001b[3' + c + ';1m' + name + ' ' + '\u001b[0m';
	
	    args[0] = prefix + args[0].split('\n').join('\n' + prefix);
	    args.push('\u001b[3' + c + 'm+' + exports.humanize(this.diff) + '\u001b[0m');
	  } else {
	    args[0] = new Date().toUTCString()
	      + ' ' + name + ' ' + args[0];
	  }
	}
	
	// node端作为exports.log日志输出方法，优先级弱于debug.log，高于console.log
	// util.format(msg,...args)函数消耗参数args转化msg中的%s字符串替换、%d数值替换、%j将json对象转化为字符串并替换
	function log() {
	  return stream.write(util.format.apply(util, arguments) + '\n');
	}
	
	// 将添加到exports.names及exports.skips中待调试及忽略的模块名namespaces赋值给process.env.DEBUG
	function save(namespaces) {
	  if (null == namespaces) {
	    delete process.env.DEBUG;
	  } else {
	    process.env.DEBUG = namespaces;
	  }
	}
	
	// 初始化以process.env.DEBUG设置exports.names及exports.skips
	function load() {
	  return process.env.DEBUG;
	}
	
	/**
	 * Copied from `node/src/node.js`.
	 *
	 * XXX: It's lame that node doesn't expose this API out-of-the-box. It also
	 * relies on the undocumented `tty_wrap.guessHandleType()` which is also lame.
	 */
	// 将日志输出到文件中？？？
	function createWritableStdioStream (fd) {
	  var stream;
	  var tty_wrap = process.binding('tty_wrap');
	
	  // Note stream._type is used for test-module-load-list.js
	
	  switch (tty_wrap.guessHandleType(fd)) {
	    case 'TTY':
	      stream = new tty.WriteStream(fd);
	      stream._type = 'tty';
	
	      // Hack to have stream not keep the event loop alive.
	      // See https://github.com/joyent/node/issues/1726
	      if (stream._handle && stream._handle.unref) {
	        stream._handle.unref();
	      }
	      break;
	
	    case 'FILE':
	      var fs = require('fs');
	      stream = new fs.SyncWriteStream(fd, { autoClose: false });
	      stream._type = 'fs';
	      break;
	
	    case 'PIPE':
	    case 'TCP':
	      var net = require('net');
	      stream = new net.Socket({
	        fd: fd,
	        readable: false,
	        writable: true
	      });
	
	      // FIXME Should probably have an option in net.Socket to create a
	      // stream from an existing fd which is writable only. But for now
	      // we'll just add this hack and set the `readable` member to false.
	      // Test: ./node test/fixtures/echo.js < /etc/passwd
	      stream.readable = false;
	      stream.read = null;
	      stream._type = 'pipe';
	
	      // FIXME Hack to have stream not keep the event loop alive.
	      // See https://github.com/joyent/node/issues/1726
	      if (stream._handle && stream._handle.unref) {
	        stream._handle.unref();
	      }
	      break;
	
	    default:
	      // Probably an error on in uv_guess_handle()
	      throw new Error('Implement me. Unknown stream file type!');
	  }
	
	  // For supporting legacy API we put the FD here.
	  stream.fd = fd;
	
	  stream._isStdio = true;
	
	  return stream;
	}
	
	// debug.inspectOpts即this.inspectOpts，据此判断带颜色输出%o、%O替换的字符串文本
	function init (debug) {
	  debug.inspectOpts = util._extend({}, exports.inspectOpts);
	}
	
	// 初始化将process.env.DEBUG各待调试及忽略模块存入exports.names及exports.skips中
	exports.enable(load());

### browser.js

浏览器端根据浏览器引擎判断是否启用颜色、参数转换处理(%j序列化json对象、%d数值替换、%s字符串替换)等。也影响electron开发环境。

	exports = module.exports = require('./debug');
	exports.log = log;
	exports.formatArgs = formatArgs;
	exports.save = save;
	exports.load = load;
	exports.useColors = useColors;
	exports.storage = 'undefined' != typeof chrome
	               && 'undefined' != typeof chrome.storage
	                  ? chrome.storage.local
	                  : localstorage();
	
	// 可选择的颜色
	exports.colors = [
	  'lightseagreen',
	  'forestgreen',
	  'goldenrod',
	  'dodgerblue',
	  'darkorchid',
	  'crimson'
	];
	
	// 根据浏览器环境开启带颜色输出日志功能，electron默认开启
	function useColors() {
	  // Electron使用chrome引擎，默认开启颜色输出
	  if (typeof window !== 'undefined' && window.process && window.process.type === 'renderer') {
	    return true;
	  }
	
	  // webkit或firefox开启带颜色输出
	  // is webkit? http://stackoverflow.com/a/16459606/376773
	  // document is undefined in react-native: https://github.com/facebook/react-native/pull/1632
	  return (typeof document !== 'undefined' && document && document.documentElement && document.documentElement.style && document.documentElement.style.WebkitAppearance) ||
	    // is firebug? http://stackoverflow.com/a/398120/376773
	    (typeof window !== 'undefined' && window && window.console && (window.console.firebug || (window.console.exception && window.console.table))) ||
	    // is firefox >= v31?
	    // https://developer.mozilla.org/en-US/docs/Tools/Web_Console#Styling_messages
	    (typeof navigator !== 'undefined' && navigator && navigator.userAgent && navigator.userAgent.toLowerCase().match(/firefox\/(\d+)/) && parseInt(RegExp.$1, 10) >= 31) ||
	    // double check webkit in userAgent just in case we are in a worker
	    (typeof navigator !== 'undefined' && navigator && navigator.userAgent && navigator.userAgent.toLowerCase().match(/applewebkit\/(\d+)/));
	}
	
	// debug(msg,...args)若msg中含有%j，将替换参数args[index]转化为json字符串
	exports.formatters.j = function(v) {
	  try {
	    return JSON.stringify(v);
	  } catch (err) {
	    return '[UnexpectedJSONParseError]: ' + err.message;
	  }
	};
	
	// 日志文案前插入命名空间，后添加时间戳，命名空间和时间戳均带颜色this.color输出，用户指定的日志文案默认颜色输出
	// 客户端带颜色打印日志为console.log(%c+str,{color:"red"})方式
	function formatArgs(args) {
	  var useColors = this.useColors;
	
	  args[0] = (useColors ? '%c' : '')
	    + this.namespace
	    + (useColors ? ' %c' : ' ')
	    + args[0]
	    + (useColors ? '%c ' : ' ')
	    + '+' + exports.humanize(this.diff);
	
	  if (!useColors) return;
	
	  var c = 'color: ' + this.color;
	  args.splice(1, 0, c, 'color: inherit')
	
	  var index = 0;
	  var lastC = 0;
	  args[0].replace(/%[a-zA-Z%]/g, function(match) {
	    if ('%%' === match) return;
	    index++;
	    if ('%c' === match) {
	      lastC = index;
	    }
	  });
	
	  args.splice(lastC, 0, c);
	}
	
	// 浏览器端以console.log作为exports.log日志输出方法，优先级弱于debug.log
	function log() {
	  return 'object' === typeof console
	    && console.log
	    && Function.prototype.apply.call(console.log, console, arguments);
	}
	
	// localStorage浏览器缓存中存储{debug:namespaces}
	function save(namespaces) {
	  try {
	    if (null == namespaces) {
	      exports.storage.removeItem('debug');
	    } else {
	      exports.storage.debug = namespaces;
	    }
	  } catch(e) {}
	}
	
	// 初始化以localStorage设置exports.names及exports.skips
	function load() {
	  var r;
	  try {
	    r = exports.storage.debug;
	  } catch(e) {}
	
	  // Electron环境下
	  if (!r && typeof process !== 'undefined' && 'env' in process) {
	    r = process.env.DEBUG;
	  }
	
	  return r;
	}
	
	// 初始化以localStorage设置exports.names及exports.skips
	exports.enable(load());
	
	// localStorage存取exports.names及exports.skips
	function localstorage() {
	  try {
	    return window.localStorage;
	  } catch (e) {}
	}

## 对外接口

### index.js

	/**
	 * Detect Electron renderer process, which is node, but we should
	 * treat as a browser.
	 */
	
	if (typeof process !== 'undefined' && process.type === 'renderer') {
	  module.exports = require('./browser.js');
	} else {
	  module.exports = require('./node.js');
	}

