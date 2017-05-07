# error-inject 1.0.0

## 概述

error-inject模块为流stream绑定error事件。

## 源码

	var Stream = require('stream');
	
	module.exports = function (stream, error) {
	  if (stream instanceof Stream
	    && !~stream.listeners('error').indexOf(error)) {
	    stream.on('error', error);
	  }
	  return stream;
	};