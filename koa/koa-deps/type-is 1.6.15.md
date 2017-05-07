# type-is 1.6.15

## 概述及使用

type-is模块用于判断请求头的content-type属性为何种mime类型，以构建koa上下文对象context的is(type)方法。

### koa中使用

	// 当请求头的Content-Type属性为'application/json'
	this.is('json', 'urlencoded'); // => 'json'
	this.is('application/json'); // => 'application/json'
	this.is('html', 'application/*'); // => 'application/json'
	this.is('html'); // => false
	
## 源码

	'use strict'
	
	// 用于序列化或反序列化消息头的content-type属性
	var typer = require('media-typer')
	// mime类型存储
	var mime = require('mime-types')
	
	module.exports = typeofrequest
	module.exports.is = typeis
	module.exports.hasBody = hasbody
	module.exports.normalize = normalize
	module.exports.match = mimeMatch
	
	// 参数value作为请求头的content-type属性，用于拾取types_传参匹配该content-type属性的首项
	// 若types_中含有通配符，且匹配该通配符，则以实际content-type属性value输出；否则以types_数组项输出
	// 若types_不传，返回value
	function typeis (value, types_) {
	  var i
	  var types = types_
	
	  var val = tryNormalizeType(value)
	
	  if (!val) {
	    return false
	  }
	
	  if (types && !Array.isArray(types)) {
	    types = new Array(arguments.length - 1)
	    for (i = 0; i < types.length; i++) {
	      types[i] = arguments[i + 1]
	    }
	  }
	
	  if (!types || !types.length) {
	    return val
	  }
	
	  var type
	  for (i = 0; i < types.length; i++) {
	    if (mimeMatch(normalize(type = types[i]), val)) {
	      return type[0] === '+' || type.indexOf('*') !== -1
	        ? val
	        : type
	    }
	  }
	
	  return false
	}
	
	// 根据请求头的'transfer-encoding'、'content-length'属性判断是否有请求内容
	function hasbody (req) {
	  return req.headers['transfer-encoding'] !== undefined ||
	    !isNaN(req.headers['content-length'])
	}
	
	// 用于构建koa上下文对象context的is(type)方法
	// 当请求头的Content-Type属性为'application/json'
	// this.is('json', 'urlencoded'); // => 'json'
	// this.is('application/json'); // => 'application/json'
	// this.is('html', 'application/*'); // => 'application/json'
	// this.is('html'); // => false
	
	// 根据请求req的消息头content-type属性，用于拾取types_传参匹配该content-type属性的首项
	// 若types_中含有通配符，且匹配该通配符，则以实际content-type属性value输出；否则以types_数组项输出
	// 若types_不传，返回请求头的content-type属性
	function typeofrequest (req, types_) {
	  var types = types_
	
	  if (!hasbody(req)) {
	    return null
	  }
	
	  if (arguments.length > 2) {
	    types = new Array(arguments.length - 1)
	    for (var i = 0; i < types.length; i++) {
	      types[i] = arguments[i + 1]
	    }
	  }
	
	  var value = req.headers['content-type']
	
	  return typeis(value, types)
	}
	
	// 将简写形式的文件扩展名或'urlencoded'、'multipart'转化为mime类型输出；mime类型原样输出
	// type以"+"起始，作为通配符处理，返回'*/*'+type，如"+json" -> "*/*+json"
	function normalize (type) {
	  if (typeof type !== 'string') {
	    return false
	  }
	
	  switch (type) {
	    case 'urlencoded':
	      return 'application/x-www-form-urlencoded'
	    case 'multipart':
	      return 'multipart/*'
	  }
	
	  if (type[0] === '+') {
	    return '*/*' + type
	  }
	
	  return type.indexOf('/') === -1
	    ? mime.lookup(type)
	    : type
	}
	
	// mime类型比较，有"*"、"*+"+type两种通配符
	function mimeMatch (expected, actual) {
	  if (expected === false) {
	    return false
	  }
	
	  var actualParts = actual.split('/')
	  var expectedParts = expected.split('/')
	
	  if (actualParts.length !== 2 || expectedParts.length !== 2) {
	    return false
	  }
	
	  if (expectedParts[0] !== '*' && expectedParts[0] !== actualParts[0]) {
	    return false
	  }
	
	  if (expectedParts[1].substr(0, 2) === '*+') {
	    return expectedParts[1].length <= actualParts[1].length + 1 &&
	      expectedParts[1].substr(1) === actualParts[1].substr(1 - expectedParts[1].length)
	  }
	
	  if (expectedParts[1] !== '*' && expectedParts[1] !== actualParts[1]) {
	    return false
	  }
	
	  return true
	}
	
	// 移除mime类型的参数，仍旧以字符串输出
	function normalizeType (value) {
	  var type = typer.parse(value)
	
	  type.parameters = undefined
	
	  return typer.format(type)
	}
	
	function tryNormalizeType (value) {
	  try {
	    return normalizeType(value)
	  } catch (err) {
	    return null
	  }
	}