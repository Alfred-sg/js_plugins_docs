# statuses 1.3.1

## 概述和使用

statuses模块即http状态码分类管理。

	// statuses含有状态码到状态码文案的正反向映射
	// 如{"100": "Continue","Continue": "100","continue": "100"}等
	var statuses=require("statuses");
	
	statues(100);// 返回100状态码
	statues("continue");// 返回100状态码
	statues(111);// 报错，默认无该状态码
	statues("xxx");// 报错，默认无该文案
	
	statuses.redirect[code];// 真值状态码code所属重定向组，如300
	statuses.empty[code];// 真值状态码code所属响应内容为空组，如204
	statuses.retry[code];// 真值状态码code所属请求待重试组，如502

## 源码

### index.js

	'use strict'
	// http状态码类型
	var codes = require('./codes.json')
	
	module.exports = status
	
	// status.codes数组形式存储状态码到状态码类型文案的映射，同时status也含有该映射
	status.codes = populateStatusesMap(status, codes)
	
	// 300、301、302、303、305、307、308重定向
	status.redirect = {
	  300: true,
	  301: true,
	  302: true,
	  303: true,
	  305: true,
	  307: true,
	  308: true
	}
	
	// 204、205、304状态码响应为空
	status.empty = {
	  204: true,
	  205: true,
	  304: true
	}
	
	// 502、503、504提示用户重试
	status.retry = {
	  502: true,
	  503: true,
	  504: true
	}
	
	// 状态码到状态码类型的映射，数组形式
	function populateStatusesMap (statuses, codes) {
	  var arr = []
	
	  Object.keys(codes).forEach(function forEachCode (code) {
	    var message = codes[code]
	    var status = Number(code)
	
	    statuses[status] = message
	    statuses[message] = status
	    statuses[message.toLowerCase()] = status
	
	    arr.push(status)
	  })
	
	  return arr
	}
	
	// 由数值状态码或状态码类型文案获取数值状态码
	function status (code) {
	  if (typeof code === 'number') {
	    if (!status[code]) throw new Error('invalid status code: ' + code)
	    return code
	  }
	
	  if (typeof code !== 'string') {
	    throw new TypeError('code must be a number or string')
	  }
	
	  // '403'
	  var n = parseInt(code, 10)
	  if (!isNaN(n)) {
	    if (!status[n]) throw new Error('invalid status code: ' + n)
	    return n
	  }
	
	  n = status[code.toLowerCase()]
	  if (!n) throw new Error('invalid status message: "' + code + '"')
	  return n
	}

### codes.json

	{
	  "100": "Continue",
	  "101": "Switching Protocols",
	  "102": "Processing",
	  "200": "OK",
	  "201": "Created",
	  "202": "Accepted",
	  "203": "Non-Authoritative Information",
	  "204": "No Content",
	  "205": "Reset Content",
	  "206": "Partial Content",
	  "207": "Multi-Status",
	  "208": "Already Reported",
	  "226": "IM Used",
	  "300": "Multiple Choices",
	  "301": "Moved Permanently",
	  "302": "Found",
	  "303": "See Other",
	  "304": "Not Modified",
	  "305": "Use Proxy",
	  "306": "(Unused)",
	  "307": "Temporary Redirect",
	  "308": "Permanent Redirect",
	  "400": "Bad Request",
	  "401": "Unauthorized",
	  "402": "Payment Required",
	  "403": "Forbidden",
	  "404": "Not Found",
	  "405": "Method Not Allowed",
	  "406": "Not Acceptable",
	  "407": "Proxy Authentication Required",
	  "408": "Request Timeout",
	  "409": "Conflict",
	  "410": "Gone",
	  "411": "Length Required",
	  "412": "Precondition Failed",
	  "413": "Payload Too Large",
	  "414": "URI Too Long",
	  "415": "Unsupported Media Type",
	  "416": "Range Not Satisfiable",
	  "417": "Expectation Failed",
	  "418": "I'm a teapot",
	  "421": "Misdirected Request",
	  "422": "Unprocessable Entity",
	  "423": "Locked",
	  "424": "Failed Dependency",
	  "425": "Unordered Collection",
	  "426": "Upgrade Required",
	  "428": "Precondition Required",
	  "429": "Too Many Requests",
	  "431": "Request Header Fields Too Large",
	  "451": "Unavailable For Legal Reasons",
	  "500": "Internal Server Error",
	  "501": "Not Implemented",
	  "502": "Bad Gateway",
	  "503": "Service Unavailable",
	  "504": "Gateway Timeout",
	  "505": "HTTP Version Not Supported",
	  "506": "Variant Also Negotiates",
	  "507": "Insufficient Storage",
	  "508": "Loop Detected",
	  "509": "Bandwidth Limit Exceeded",
	  "510": "Not Extended",
	  "511": "Network Authentication Required"
	}