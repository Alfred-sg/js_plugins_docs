# content-type 1.0.2

## 概述

content-type模块用于序列化及反序列化消息头的content-type属性。

## 源码

	'use strict'
	
	// 获取参数
	var paramRegExp = /; *([!#$%&'\*\+\-\.\^_`\|~0-9A-Za-z]+) *= *("(?:[\u000b\u0020\u0021\u0023-\u005b\u005d-\u007e\u0080-\u00ff]|\\[\u000b\u0020-\u00ff])*"|[!#$%&'\*\+\-\.\^_`\|~0-9A-Za-z]+) */g
	// 用于校验参数的值
	var textRegExp = /^[\u000b\u0020-\u007e\u0080-\u00ff]+$/
	// 用于校验参数的键、值
	var tokenRegExp = /^[!#$%&'\*\+\-\.\^_`\|~0-9A-Za-z]+$/
	
	var qescRegExp = /\\([\u000b\u0020-\u00ff])/g
	
	var quoteRegExp = /([\\"])/g
	
	// 校验消息头的contentType属性
	var typeRegExp = /^[!#$%&'\*\+\-\.\^_`\|~0-9A-Za-z]+\/[!#$%&'\*\+\-\.\^_`\|~0-9A-Za-z]+$/
	
	exports.format = format
	exports.parse = parse
	
	// 将消息头的content-type属性从对象形式拼接为字符串形式'image/svg+xml;charset=utf-8'
	function format(obj) {
	  if (!obj || typeof obj !== 'object') {
	    throw new TypeError('argument obj is required')
	  }
	
	  var parameters = obj.parameters
	  var type = obj.type
	
	  if (!type || !typeRegExp.test(type)) {
	    throw new TypeError('invalid type')
	  }
	
	  var string = type
	
	  if (parameters && typeof parameters === 'object') {
	    var param
	    var params = Object.keys(parameters).sort()
	
	    for (var i = 0; i < params.length; i++) {
	      param = params[i]
	
	      if (!tokenRegExp.test(param)) {
	        throw new TypeError('invalid parameter name')
	      }
	
	      string += '; ' + param + '=' + qstring(parameters[param])
	    }
	  }
	
	  return string
	}
	
	// 将消息头的content-type属性从字符串解析为对象形式{type,parameters:{key:value}}形式
	function parse(string) {
	  if (!string) {
	    throw new TypeError('argument string is required')
	  }
	
	  if (typeof string === 'object') {
	    string = getcontenttype(string)
	
	    if (typeof string !== 'string') {
	      throw new TypeError('content-type header is missing from object');
	    }
	  }
	
	  if (typeof string !== 'string') {
	    throw new TypeError('argument string is required to be a string')
	  }
	
	  var index = string.indexOf(';')
	  var type = index !== -1
	    ? string.substr(0, index).trim()
	    : string.trim()
	
	  if (!typeRegExp.test(type)) {
	    throw new TypeError('invalid media type')
	  }
	
	  var key
	  var match
	  var obj = new ContentType(type.toLowerCase())
	  var value
	
	  paramRegExp.lastIndex = index
	
	  while (match = paramRegExp.exec(string)) {
	    if (match.index !== index) {
	      throw new TypeError('invalid parameter format')
	    }
	
	    index += match[0].length
	    key = match[1].toLowerCase()
	    value = match[2]
	
	    if (value[0] === '"') {
	      value = value
	        .substr(1, value.length - 2)
	        .replace(qescRegExp, '$1')
	    }
	
	    obj.parameters[key] = value
	  }
	
	  if (index !== -1 && index !== string.length) {
	    throw new TypeError('invalid parameter format')
	  }
	
	  return obj
	}
	
	// obj为请求req或响应res对象
	function getcontenttype(obj) {
	  if (typeof obj.getHeader === 'function') {
	    return obj.getHeader('content-type')
	  }
	
	  if (typeof obj.headers === 'object') {
	    return obj.headers && obj.headers['content-type']
	  }
	}
	
	// 校验参数的值，并获得该值
	function qstring(val) {
	  var str = String(val)
	
	  if (tokenRegExp.test(str)) {
	    return str
	  }
	
	  if (str.length > 0 && !textRegExp.test(str)) {
	    throw new TypeError('invalid parameter value')
	  }
	
	  return '"' + str.replace(quoteRegExp, '\\$1') + '"'
	}
	
	function ContentType(type) {
	  this.parameters = Object.create(null)
	  this.type = type
	}