# destroy 1.0.4

## 概述

destroy用于销毁流stream。koa模块中使用用流为响应内容注入值后，予以销毁流。

## 源码
	
	'use strict'
	
	var ReadStream = require('fs').ReadStream
	var Stream = require('stream')
	
	module.exports = destroy
	
	function destroy(stream) {
	  if (stream instanceof ReadStream) {
	    return destroyReadStream(stream)
	  }
	
	  if (!(stream instanceof Stream)) {
	    return stream
	  }
	
	  if (typeof stream.destroy === 'function') {
	    stream.destroy()
	  }
	
	  return stream
	}
	
	function destroyReadStream(stream) {
	  stream.destroy()
	
	  if (typeof stream.close === 'function') {
	    // node.js core bug work-around
	    stream.on('open', onOpenClose)
	  }
	
	  return stream
	}

function onOpenClose() {
  if (typeof this.fd === 'number') {
    // actually close down the fd
    this.close()
  }
}
