# vary 1.1.1

## 概述及使用

vary模块用于设置响应头的vary属性。

	var http = require('http')
	var vary = require('vary')
	
	http.createServer(function onRequest (req, res) {
	  vary(res, 'User-Agent')
	
	  var ua = req.headers['user-agent'] || ''
	  var isMobile = /mobi|android|touch|mini/i.test(ua)
	
	  res.setHeader('Content-Type', 'text/html')
	  res.end('You are (probably) ' + (isMobile ? '' : 'not ') + 'a mobile user')
	})
	
## 源码

	'use strict'
	
	module.exports = vary
	module.exports.append = append
	
	var ARRAY_SPLIT_REGEXP = / *, */
	
	// 校验单个响应头的vary属性
	var FIELD_NAME_REGEXP = /^[!#$%&'*+\-.^_`|~0-9A-Za-z]+$/
	
	// 参数header为响应头的vary属性，以字符串添加field后返回
	function append (header, field) {
	  if (typeof header !== 'string') {
	    throw new TypeError('header argument is required')
	  }
	
	  if (!field) {
	    throw new TypeError('field argument is required')
	  }
	
	  var fields = !Array.isArray(field)
	    ? parse(String(field))
	    : field
	
	  for (var j = 0; j < fields.length; j++) {
	    if (!FIELD_NAME_REGEXP.test(fields[j])) {
	      throw new TypeError('field argument contains an invalid header name')
	    }
	  }
	
	  if (header === '*') {
	    return header
	  }
	
	  var val = header
	  var vals = parse(header.toLowerCase())
	
	  // 含有通配符，只须设置单个通配符"*"
	  if (fields.indexOf('*') !== -1 || vals.indexOf('*') !== -1) {
	    return '*'
	  }
	
	  for (var i = 0; i < fields.length; i++) {
	    var fld = fields[i].toLowerCase()
	
	    if (vals.indexOf(fld) === -1) {
	      vals.push(fld)
	      val = val
	        ? val + ', ' + fields[i]
	        : fields[i]
	    }
	  }
	
	  return val
	}
	
	function parse (header) {
	  return header.trim().split(ARRAY_SPLIT_REGEXP)
	}
	
	// 响应头的vary属性添加field后，重新设置响应头的vary属性
	function vary (res, field) {
	  if (!res || !res.getHeader || !res.setHeader) {
	    throw new TypeError('res argument is required')
	  }
	
	  var val = res.getHeader('Vary') || ''
	  var header = Array.isArray(val)
	    ? val.join(', ')
	    : String(val)
	
	  if ((val = append(header, field))) {
	    res.setHeader('Vary', val)
	  }
	}