# media-type 0.3.0

## 概述

media-type模块同content-type模块，用于序列化、反序列化消息头的content-type属性；media-type模块颗粒度更细。

## 源码

	var paramRegExp = /; *([!#$%&'\*\+\-\.0-9A-Z\^_`a-z\|~]+) *= *("(?:[ !\u0023-\u005b\u005d-\u007e\u0080-\u00ff]|\\[\u0020-\u007e])*"|[!#$%&'\*\+\-\.0-9A-Z\^_`a-z\|~]+) */g;
	var textRegExp = /^[\u0020-\u007e\u0080-\u00ff]+$/
	var tokenRegExp = /^[!#$%&'\*\+\-\.0-9A-Z\^_`a-z\|~]+$/
	
	var qescRegExp = /\\([\u0000-\u007f])/g;
	
	var quoteRegExp = /([\\"])/g;
	
	var subtypeNameRegExp = /^[A-Za-z0-9][A-Za-z0-9!#$&^_.-]{0,126}$/
	var typeNameRegExp = /^[A-Za-z0-9][A-Za-z0-9!#$&^_-]{0,126}$/
	var typeRegExp = /^ *([A-Za-z0-9][A-Za-z0-9!#$&^_-]{0,126})\/([A-Za-z0-9][A-Za-z0-9!#$&^_.+-]{0,126}) *$/;
	
	// 将消息头的content-type属性从对象形式拼接为字符串形式'image/svg+xml;charset=utf-8'
	exports.format = format
	// 将消息头的content-type属性从字符串解析为对象形式{type,subtype,suffix,parameters:{key:value}}形式
	// 较content-type模块解析形式{type,parameters:{key:value}}，media-type颗粒度更高
	exports.parse = parse
	
	function format(obj) {
	  if (!obj || typeof obj !== 'object') {
	    throw new TypeError('argument obj is required')
	  }
	
	  var parameters = obj.parameters
	  var subtype = obj.subtype
	  var suffix = obj.suffix
	  var type = obj.type
	
	  if (!type || !typeNameRegExp.test(type)) {
	    throw new TypeError('invalid type')
	  }
	
	  if (!subtype || !subtypeNameRegExp.test(subtype)) {
	    throw new TypeError('invalid subtype')
	  }
	
	  var string = type + '/' + subtype
	
	  if (suffix) {
	    if (!typeNameRegExp.test(suffix)) {
	      throw new TypeError('invalid suffix')
	    }
	
	    string += '+' + suffix
	  }
	
	  // append parameters
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
	
	function parse(string) {
	  if (!string) {
	    throw new TypeError('argument string is required')
	  }
	
	  if (typeof string === 'object') {
	    string = getcontenttype(string)
	  }
	
	  if (typeof string !== 'string') {
	    throw new TypeError('argument string is required to be a string')
	  }
	
	  var index = string.indexOf(';')
	  var type = index !== -1
	    ? string.substr(0, index)
	    : string
	
	  var key
	  var match
	  var obj = splitType(type)
	  var params = {}
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
	
	    params[key] = value
	  }
	
	  if (index !== -1 && index !== string.length) {
	    throw new TypeError('invalid parameter format')
	  }
	
	  obj.parameters = params
	
	  return obj
	}
	
	function getcontenttype(obj) {
	  if (typeof obj.getHeader === 'function') {
	    // res-like
	    return obj.getHeader('content-type')
	  }
	
	  if (typeof obj.headers === 'object') {
	    // req-like
	    return obj.headers && obj.headers['content-type']
	  }
	}
	
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
	
	function splitType(string) {
	  var match = typeRegExp.exec(string.toLowerCase())
	
	  if (!match) {
	    throw new TypeError('invalid media type')
	  }
	
	  var type = match[1]
	  var subtype = match[2]
	  var suffix
	
	  var index = subtype.lastIndexOf('+')
	  if (index !== -1) {
	    suffix = subtype.substr(index + 1)
	    subtype = subtype.substr(0, index)
	  }
	
	  var obj = {
	    type: type,
	    subtype: subtype,
	    suffix: suffix
	  }
	
	  return obj
	}