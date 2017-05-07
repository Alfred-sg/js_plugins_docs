# content-disposition 0.5.2

## 概述及使用

content-disposition模块用于通过文件名创建响应头content-disposition属性字符串，或者将响应头content-disposition属性字符串解析为对象形式{type,	params}。用于构建koa中间件函数上下文对象context的attachment方法，this.attachment(filename)使filename文件在客户端表现为可下载。

	var contentDisposition = require('content-disposition')
	var destroy = require('destroy')
	var http = require('http')
	var onFinished = require('on-finished')
	
	var filePath = '/path/to/public/plans.pdf'
	
	http.createServer(function onRequest(req, res) {
	  // set headers
	  res.setHeader('Content-Type', 'application/pdf')
	  res.setHeader('Content-Disposition', contentDisposition(filePath))
	
	  // send file
	  var stream = fs.createReadStream(filePath)
	  stream.pipe(res)
	  onFinished(res, function (err) {
	    destroy(stream)
	  })
	})
	
## 源码
	
	'use strict'
	
	module.exports = contentDisposition
	module.exports.parse = parse
	
	var basename = require('path').basename// 获取文件名，含或不含扩展名
	
	var ENCODE_URL_ATTR_CHAR_REGEXP = /[\x00-\x20"'()*,/:;<=>?@[\\\]{}\x7f]/g // eslint-disable-line no-control-regex
	
	var HEX_ESCAPE_REGEXP = /%[0-9A-Fa-f]{2}/
	var HEX_ESCAPE_REPLACE_REGEXP = /%([0-9A-Fa-f]{2})/g
	
	// 校验fallback
	var NON_LATIN1_REGEXP = /[^\x20-\x7e\xa0-\xff]/g
	
	var QESC_REGEXP = /\\([\u0000-\u007f])/g
	
	var QUOTE_REGEXP = /([\\"])/g
	
	// 参数匹配正则
	var PARAM_REGEXP = /;[\x09\x20]*([!#$%&'*+.0-9A-Z^_`a-z|~-]+)[\x09\x20]*=[\x09\x20]*("(?:[\x20!\x23-\x5b\x5d-\x7e\x80-\xff]|\\[\x20-\x7e])*"|[!#$%&'*+.0-9A-Z^_`a-z|~-]+)[\x09\x20]*/g // eslint-disable-line no-control-regex
	// 校验文件名
	var TEXT_REGEXP = /^[\x20-\x7e\x80-\xff]+$/
	// 校验type
	var TOKEN_REGEXP = /^[!#$%&'*+.0-9A-Z^_`a-z|~-]+$/
	
	// 参数值匹配正则
	var EXT_VALUE_REGEXP = /^([A-Za-z0-9!#$%&+\-^_`{}~]+)'(?:[A-Za-z]{2,3}(?:-[A-Za-z]{3}){0,3}|[A-Za-z]{4,8}|)'((?:%[0-9A-Fa-f]{2}|[A-Za-z0-9!#$&+.^_`|~-])+)$/
	
	/**
	 * 响应头content-disposition属性字符串匹配正则
	 *
	 * disposition-type = "inline" | "attachment" | disp-ext-type
	 * disp-ext-type    = token
	 * disposition-parm = filename-parm | disp-ext-parm
	 * filename-parm    = "filename" "=" value
	 *                  | "filename*" "=" ext-value
	 * disp-ext-parm    = token "=" value
	 *                  | ext-token "=" ext-value
	 * ext-token        = <the characters in token, followed by "*">
	 */
	var DISPOSITION_TYPE_REGEXP = /^([!#$%&'*+.0-9A-Z^_`a-z|~-]+)[\x09\x20]*(?:$|;)/ // eslint-disable-line no-control-regex
	
	// 响应头的content-disposition属性设为"attachment"时提示下载，而不是使用网页打开
	// contentDisposition(filename,{type,fallback})获取将要设置为消息头content-disposition属性的字符串
	//    如'attachment; filename="EURO rates.txt"; filename*=UTF-8\'\'%e2%82%ac%20rates.txt'
	// 参数options.fallback用于重设下载文件名
	function contentDisposition (filename, options) {
	  var opts = options || {}
	
	  var type = opts.type || 'attachment'
	
	  var params = createparams(filename, opts.fallback)
	
	  return format(new ContentDisposition(type, params))
	}
	
	function createparams (filename, fallback) {
	  if (filename === undefined) {
	    return
	  }
	
	  var params = {}
	
	  if (typeof filename !== 'string') {
	    throw new TypeError('filename must be a string')
	  }
	
	  // fallback defaults to true
	  if (fallback === undefined) {
	    fallback = true
	  }
	
	  if (typeof fallback !== 'string' && typeof fallback !== 'boolean') {
	    throw new TypeError('fallback must be a string or boolean')
	  }
	
	  if (typeof fallback === 'string' && NON_LATIN1_REGEXP.test(fallback)) {
	    throw new TypeError('fallback must be ISO-8859-1 string')
	  }
	
	  var name = basename(filename)// 获取文件名，含扩展名
	
	  var isQuotedString = TEXT_REGEXP.test(name)
	
	  var fallbackName = typeof fallback !== 'string'
	    ? fallback && getlatin1(name)
	    : basename(fallback)
	  var hasFallback = typeof fallbackName === 'string' && fallbackName !== name
	
	  if (hasFallback || !isQuotedString || HEX_ESCAPE_REGEXP.test(name)) {
	    params['filename*'] = name
	  }
	
	  if (isQuotedString || hasFallback) {
	    params.filename = hasFallback
	      ? fallbackName
	      : name
	  }
	
	  return params
	}
	
	function format (obj) {
	  var parameters = obj.parameters
	  var type = obj.type
	
	  if (!type || typeof type !== 'string' || !TOKEN_REGEXP.test(type)) {
	    throw new TypeError('invalid type')
	  }
	
	  var string = String(type).toLowerCase()
	
	  if (parameters && typeof parameters === 'object') {
	    var param
	    var params = Object.keys(parameters).sort()
	
	    for (var i = 0; i < params.length; i++) {
	      param = params[i]
	
	      var val = param.substr(-1) === '*'
	        ? ustring(parameters[param])// 转化为utf-8字符
	        : qstring(parameters[param])// 加双引号
	
	      string += '; ' + param + '=' + val
	    }
	  }
	
	  return string
	}
	
	function decodefield (str) {
	  var match = EXT_VALUE_REGEXP.exec(str)
	
	  if (!match) {
	    throw new TypeError('invalid extended field value')
	  }
	
	  var charset = match[1].toLowerCase()
	  var encoded = match[2]
	  var value
	
	  var binary = encoded.replace(HEX_ESCAPE_REPLACE_REGEXP, pdecode)
	
	  switch (charset) {
	    case 'iso-8859-1':
	      value = getlatin1(binary)
	      break
	    case 'utf-8':
	      value = new Buffer(binary, 'binary').toString('utf8')
	      break
	    default:
	      throw new TypeError('unsupported charset in extended field')
	  }
	
	  return value
	}
	
	// 将Unicode字符转化为ISO-8859-1字符
	function getlatin1 (val) {
	  return String(val).replace(NON_LATIN1_REGEXP, '?')
	}
	
	// 将响应头content-disposition属性字符解析为对象形式{type,params}
	function parse (string) {
	  if (!string || typeof string !== 'string') {
	    throw new TypeError('argument string is required')
	  }
	
	  var match = DISPOSITION_TYPE_REGEXP.exec(string)
	
	  if (!match) {
	    throw new TypeError('invalid type format')
	  }
	
	  var index = match[0].length
	  var type = match[1].toLowerCase()
	
	  var key
	  var names = []
	  var params = {}
	  var value
	
	  index = PARAM_REGEXP.lastIndex = match[0].substr(-1) === ';'
	    ? index - 1
	    : index
	
	  while ((match = PARAM_REGEXP.exec(string))) {
	    if (match.index !== index) {
	      throw new TypeError('invalid parameter format')
	    }
	
	    index += match[0].length
	    key = match[1].toLowerCase()
	    value = match[2]
	
	    if (names.indexOf(key) !== -1) {
	      throw new TypeError('invalid duplicate parameter')
	    }
	
	    names.push(key)
	
	    if (key.indexOf('*') + 1 === key.length) {
	      key = key.slice(0, -1)
	      value = decodefield(value)
	
	      params[key] = value
	      continue
	    }
	
	    if (typeof params[key] === 'string') {
	      continue
	    }
	
	    if (value[0] === '"') {
	      // remove quotes and escapes
	      value = value
	        .substr(1, value.length - 2)
	        .replace(QESC_REGEXP, '$1')
	    }
	
	    params[key] = value
	  }
	
	  if (index !== -1 && index !== string.length) {
	    throw new TypeError('invalid parameter format')
	  }
	
	  return new ContentDisposition(type, params)
	}
	
	function pdecode (str, hex) {
	  return String.fromCharCode(parseInt(hex, 16))
	}
	
	function pencode (char) {
	  var hex = String(char)
	    .charCodeAt(0)
	    .toString(16)
	    .toUpperCase()
	  return hex.length === 1
	    ? '%0' + hex
	    : '%' + hex
	}
	
	// 加双引号
	function qstring (val) {
	  var str = String(val)
	
	  return '"' + str.replace(QUOTE_REGEXP, '\\$1') + '"'
	}
	
	// 转化为utf-8字符
	function ustring (val) {
	  var str = String(val)
	
	  var encoded = encodeURIComponent(str)
	    .replace(ENCODE_URL_ATTR_CHAR_REGEXP, pencode)
	
	  return 'UTF-8\'\'' + encoded
	}
	
	function ContentDisposition (type, parameters) {
	  this.type = type
	  this.parameters = parameters
	}