# fresh 0.5.0

## 概述

fresh模块用于判断单次请求是否需要重新获取服务器端资源，还是启用本地缓存。

## 源码

	'use strict'
	
	var CACHE_CONTROL_NO_CACHE_REGEXP = /(?:^|,)\s*?no-cache\s*?(?:,|$)/
	
	var TOKEN_LIST_REGEXP = / *, */
	
	module.exports = fresh
	
	// 用于判断单次请求是否需要重新获取服务器端资源，还是启用本地缓存
	function fresh (reqHeaders, resHeaders) {
	  // 用于判断客户端本地缓存是否服务器端最新修改资源
	  // 值为服务器端返回资源的响应头'last-modified'属性，客户端缓存的请求资源的修改时间
	  // 如未修改返回304，启用客户端本地缓存；若修改，重新发送服务器端资源
	  var modifiedSince = reqHeaders['if-modified-since']
	
	  // 用于判断客户端本地缓存是否服务器端最新修改资源，值为服务器端返回资源的响应头etag属性
	  // 如未修改返回304，启用客户端本地缓存；若修改，重新发送服务器端资源
	  var noneMatch = reqHeaders['if-none-match']
	
	  if (!modifiedSince && !noneMatch) {
	    return false
	  }
	
	  // 消息头"cache-control"指定客户端的缓存机制，值为"private"、"no-cache"、"max-age"、"must-revalidate"等，默认为"private"
	  // 打开新窗口，"private"、"no-cache"、"must-revalidate"重新获取资源；"max-age"在特定时间内使用本地缓存
	  // 地址栏回车，"private"、"must-revalidate"第一次请求时重新获取资源；"max-age"在特定时间内使用本地缓存
	  //    "no-cache"每次回车重新获取资源
	  // 回退，"private"、"must-revalidate"、"max-age"不重新请求资源；"no-cache"重新获取资源
	  // 刷新，无论任何设置均重新获取资源
	  var cacheControl = reqHeaders['cache-control']
	  if (cacheControl && CACHE_CONTROL_NO_CACHE_REGEXP.test(cacheControl)) {
	    return false
	  }
	
	  if (noneMatch && noneMatch !== '*') {
	    var etag = resHeaders['etag']
	    var etagStale = !etag || noneMatch.split(TOKEN_LIST_REGEXP).every(function (match) {
	      return match !== etag && match !== 'W/' + etag && 'W/' + match !== etag
	    })
	
	    if (etagStale) {
	      return false
	    }
	  }
	
	  if (modifiedSince) {
	    var lastModified = resHeaders['last-modified']
	    var modifiedStale = !lastModified || Date.parse(lastModified) > Date.parse(modifiedSince)
	
	    if (modifiedStale) {
	      return false
	    }
	  }
	
	  return true
	}

## 参考

* [HTTP消息头详解](http://blog.csdn.net/huangjin0507/article/details/52170460)
* [HTTP响应头和请求头信息对照表](http://tools.jb51.net/table/http_header)