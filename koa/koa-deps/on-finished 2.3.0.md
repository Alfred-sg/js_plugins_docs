# on-finished 2.3.0

## 概述及使用

on-finished用于监听请求或响应终结时，触发执行系列回调函数；以及socket对象报错或暂停时，触发执行系列回调函数。

	var onFinished=require("on-finished");
	onFinished(res,listener1);
	onFinished(res,listener2);
	// 监听响应res的"end"、"finish"事件，事件触发时，依次执行listener1、listener2，以err或null为首参，次参即res对象
	// 若为socket连接，同时监听res.socket的"error"、"close"，回调函数同上
	
## 源码

	'use strict'
	
	module.exports = onFinished
	module.exports.isFinished = isFinished
	
	// first([ee,...events],done)为事件对象ee绑定...events事件，事件触发时执行done.apply(err,ee,event,args)
	// 返回值thunk(fn)用于重设回调函数done；thunk.cancel解绑事件、清空绑定函数
	var first = require('ee-first')
	
	var defer = typeof setImmediate === 'function'
	  ? setImmediate
	  : function(fn){ process.nextTick(fn.bind.apply(fn, arguments)) }
	
	// msg请求或响应已终结，立即执行listener(null,msg)
	// 未终结，监听msg的'end'、'finish'事件，事件触发时，执行listener(err,msg)回调
	// onFinished(msg,listener)可执行多次，添加多个listener，事件触发时，顺序调用listener
	function onFinished(msg, listener) {
	  if (isFinished(msg) !== false) {
	    defer(listener, null, msg)
	    return msg
	  }
	
	  // 将listener添加到msg.__onFinished.queue队列中，'end'、'finish'事件触发时调用执行
	  attachListener(msg, listener)
	
	  return msg
	}
	
	// 判断msg即请求或响应是否完结，响应判断finished熟悉感或socket不可写，请求判断upgrade属性或socket不可读
	function isFinished(msg) {
	  var socket = msg.socket
	
	  if (typeof msg.finished === 'boolean') {
	    return Boolean(msg.finished || (socket && !socket.writable))
	  }
	
	  if (typeof msg.complete === 'boolean') {
	    return Boolean(msg.upgrade || !socket || !socket.readable || (msg.complete && !msg.readable))
	  }
	
	  return undefined
	}
	
	// 监听msg或msg.socket对象，'end'、'finish'事件触发时，执行callback即msg.__onFinished
	// 进而调用msg.__onFinished.queue中的各绑定函数fn(err,msg)
	function attachFinishedListener(msg, callback) {
	  var eeMsg
	  var eeSocket
	  var finished = false
	
	  function onFinish(error) {
	    eeMsg.cancel()// 解绑msg对象的事件及清空绑定函数
	    eeSocket.cancel()// 解绑msg或msg.socket对象的事件及清空绑定函数
	
	    finished = true
	    callback(error)
	  }
	
	  // 监听msg对象'end'、'finish'事件，事件触发时解绑事件，执行onFinish.apply(err,ee,event,args)回调
	  eeMsg = eeSocket = first([[msg, 'end', 'finish']], onFinish)
	
	  function onSocket(socket) {
	    msg.removeListener('socket', onSocket)
	
	    if (finished) return
	    if (eeMsg !== eeSocket) return// socket已分派，无需再监听msg.socket对象'error'、'close'事件
	
	    // 监听socket对象'error'、'close'事件，事件触发时解绑事件，执行onFinish.apply(err,ee,event,args)回调
	    eeSocket = first([[socket, 'error', 'close']], onFinish)
	  }
	
	  // msg.socket为真值时，监听msg.socket对象的'error'、'close'事件
	  if (msg.socket) {
	    onSocket(msg.socket)
	    return
	  }
	
	  // 等待socket被分派；被分派时，开启监听msg.socket对象'error'、'close'事件
	  msg.on('socket', onSocket)
	
	  if (msg.socket === undefined) {
	    // node.js 0.8 patch
	    patchAssignSocket(msg, onSocket)
	  }
	}
	
	// 为msg对象添加msg.__onFinished监听函数，msg.__onFinished.queue存储绑定函数
	function attachListener(msg, listener) {
	  var attached = msg.__onFinished
	
	  if (!attached || !attached.queue) {
	    attached = msg.__onFinished = createListener(msg)// 构建监听函数，存储于msg.__onFinished中
	    attachFinishedListener(msg, attached)
	  }
	
	  attached.queue.push(listener)
	}
	
	// 构建listener监听函数，赋值给msg.__onFinished
	// listener的queue属性用于存储绑定函数，listener调用时执行queue属性中存储的绑定函数
	function createListener(msg) {
	  function listener(err) {
	    if (msg.__onFinished === listener) msg.__onFinished = null
	    if (!listener.queue) return
	
	    var queue = listener.queue
	    listener.queue = null
	
	    for (var i = 0; i < queue.length; i++) {
	      queue[i](err, msg)
	    }
	  }
	
	  listener.queue = []
	
	  return listener
	}
	
	// node 0.8派发msg.socket无事件系统支持，直接调动callback即onSocket函数为msg.socket绑定事件
	function patchAssignSocket(res, callback) {
	  var assignSocket = res.assignSocket
	
	  if (typeof assignSocket !== 'function') return
	
	  // res.on('socket', callback) is broken in 0.8
	  res.assignSocket = function _assignSocket(socket) {
	    assignSocket.call(this, socket)
	    callback(socket)
	  }
	}
