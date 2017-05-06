# parseurl 1.3.1

## 概述

parseurl模块用于将请求对象的路径解析为{path, href, pathname, search, query}形式，各属性值均为字符串，用于后续处理。

## 源码

	'use strict'
	
	var url = require('url')
	var parse = url.parse
	var Url = url.Url
	
	var simplePathRegExp = /^(\/\/?(?!\/)[^\?#\s]*)(\?[^#\s]*)?$/
	
	// parseUrl(req)将req.url解析为对象形式{path,href,pathname,search,query}
	// path、href属性为完整路径，pathname路径名，search含?的查询参数字符串，query不含?的查询参数字符串
	module.exports = parseurl
	
	// original(req)将req.originalUrl解析为对象形式{path,href,pathname,search,query}
	module.exports.original = originalurl
	
	function parseurl(req) {
	  var url = req.url
	
	  if (url === undefined) {
	    return undefined
	  }
	
	  var parsed = req._parsedUrl
	
	  if (fresh(url, parsed)) {
	    return parsed
	  }
	
	  parsed = fastparse(url)
	  parsed._raw = url
	
	  return req._parsedUrl = parsed
	};
	
	function originalurl(req) {
	  var url = req.originalUrl
	
	  if (typeof url !== 'string') {
	    return parseurl(req)
	  }
	
	  var parsed = req._parsedOriginalUrl
	
	  if (fresh(url, parsed)) {
	    return parsed
	  }
	
	  parsed = fastparse(url)
	  parsed._raw = url
	
	  return req._parsedOriginalUrl = parsed
	};
	
	function fastparse(str) {
	  var simplePath = typeof str === 'string' && simplePathRegExp.exec(str)
	
	  if (simplePath) {
	    var pathname = simplePath[1]
	    var search = simplePath[2] || null
	    var url = Url !== undefined
	      ? new Url()
	      : {}
	    url.path = str
	    url.href = str
	    url.pathname = pathname
	    url.search = search
	    url.query = search && search.substr(1)
	
	    return url
	  }
	
	  return parse(str)
	}
	
	// 校验req._parsedUrl对象是否url解析后的路径对象
	function fresh(url, parsedUrl) {
	  return typeof parsedUrl === 'object'
	    && parsedUrl !== null
	    && (Url === undefined || parsedUrl instanceof Url)
	    && parsedUrl._raw === url
	}