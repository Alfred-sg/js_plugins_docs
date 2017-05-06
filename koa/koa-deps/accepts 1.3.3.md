# accepts 1.3.3

## 概述及使用

accepts用于获取请求头的accept-*属性，转化为数组后输出；或者由传参获取最佳匹配类型，用于设置响应内容的传输形式。

	var accepts = require('accepts')
	var http = require('http')
	
	function app(req, res) {
	  var accept = accepts(req)
	
	  // 获取响应内容的最佳传输格式
	  switch(accept.type(['json', 'html'])) {
	    case 'json':
	      res.setHeader('Content-Type', 'application/json')
	      res.write('{"hello":"world!"}')
	      break
	    case 'html':
	      res.setHeader('Content-Type', 'text/html')
	      res.write('<b>hello, world!</b>')
	      break
	    default:
	      res.setHeader('Content-Type', 'text/plain')
	      res.write('hello, world!')
	      break
	  }
	
	  res.end()
	}
	
## 源码

	'use strict'
	
	// 判断请求头的accept-*属性或判断传参是否匹配请求头的accept-*属性
	var Negotiator = require('negotiator')
	// 根据文件名或扩展名或mime类型获取包含编码属性的mime类型
	var mime = require('mime-types')
	
	module.exports = Accepts
	
	function Accepts(req) {
	  if (!(this instanceof Accepts))
	    return new Accepts(req)
	
	  this.headers = req.headers
	  this.negotiator = new Negotiator(req)
	}
	
	// type()获取请求头的accept-type属性，数组形式如['text/*', ' application/json']
	// type(types)获取types中匹配请求头的accept-type属性且优先级最高的type，转化为传参类型输出
	//    如传参为mime类型，输出mime类型，即this.types('text/html')=> "text/html"
	//    如传参为文件扩展名，输出文件扩展名，即this.types(['html', 'json'])=> "json"
	// 参数可以是多参数字符串形式，或者单参数数组形式
	Accepts.prototype.type =
	Accepts.prototype.types = function (types_) {
	  var types = types_
	
	  if (types && !Array.isArray(types)) {
	    types = new Array(arguments.length)
	    for (var i = 0; i < types.length; i++) {
	      types[i] = arguments[i]
	    }
	  }
	
	  if (!types || types.length === 0) {
	    return this.negotiator.mediaTypes()
	  }
	
	  if (!this.headers.accept) return types[0];
	  var mimes = types.map(extToMime);// 将文件扩展名转化为mime类型
	  var accepts = this.negotiator.mediaTypes(mimes.filter(validMime));
	  var first = accepts[0];
	  if (!first) return false;
	  return types[mimes.indexOf(first)];// first为mime类型，返回值为mime类型或文件扩展名形式
	}
	
	// encoding()获取请求头的accept-encoding属性，数组形式如['gzip', 'deflate']
	// encoding(encodings)获取encodings中匹配请求头的accept-encoding属性且优先级最高的encoding，如"gzip"
	Accepts.prototype.encoding =
	Accepts.prototype.encodings = function (encodings_) {
	  var encodings = encodings_
	
	  if (encodings && !Array.isArray(encodings)) {
	    encodings = new Array(arguments.length)
	    for (var i = 0; i < encodings.length; i++) {
	      encodings[i] = arguments[i]
	    }
	  }
	
	  if (!encodings || encodings.length === 0) {
	    return this.negotiator.encodings()
	  }
	
	  return this.negotiator.encodings(encodings)[0] || false
	}
	
	// charset()获取请求头的accept-charset属性，数组形式如['utf-8', 'utf-7', 'iso-8859-1']
	// charset(charsets)获取charsets中匹配请求头的accept-charset属性且优先级最高的charset，如"utf-8"
	Accepts.prototype.charset =
	Accepts.prototype.charsets = function (charsets_) {
	  var charsets = charsets_
	
	  if (charsets && !Array.isArray(charsets)) {
	    charsets = new Array(arguments.length)
	    for (var i = 0; i < charsets.length; i++) {
	      charsets[i] = arguments[i]
	    }
	  }
	
	  if (!charsets || charsets.length === 0) {
	    return this.negotiator.charsets()
	  }
	
	  return this.negotiator.charsets(charsets)[0] || false
	}
	
	// language()获取请求头的accept-language属性，数组形式如['es', 'pt', 'en']
	// language(languages)获取languages中匹配请求头的accept-language属性且优先级最高的language，如"es"
	Accepts.prototype.lang =
	Accepts.prototype.langs =
	Accepts.prototype.language =
	Accepts.prototype.languages = function (languages_) {
	  var languages = languages_
	
	  if (languages && !Array.isArray(languages)) {
	    languages = new Array(arguments.length)
	    for (var i = 0; i < languages.length; i++) {
	      languages[i] = arguments[i]
	    }
	  }
	
	  if (!languages || languages.length === 0) {
	    return this.negotiator.languages()
	  }
	
	  return this.negotiator.languages(languages)[0] || false
	}
	
	// 将文件扩展名转化为mime类型
	function extToMime(type) {
	  return type.indexOf('/') === -1
	    ? mime.lookup(type)
	    : type
	}
	
	function validMime(type) {
	  return typeof type === 'string';
	}