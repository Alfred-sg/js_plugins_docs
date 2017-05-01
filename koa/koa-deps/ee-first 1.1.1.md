# ee-first 1.1.1

## 概述及使用

ee-first模块为事件对象绑定系列事件及回调，当首个事件触发执行时，解绑所有事件及清空绑定函数。

	var first=require("ee-first");
	first([res,"end","finish"],done);
	// 监听res对象的"end"、"finish"事件，当"end"事件率先触发时，执行done(err,res,"end",args)回调，并解绑事件、清除绑定函数

## 源码

	'use strict'
	
	module.exports = first
	
	// first([[ee,...events]],done)为事件对象ee绑定...events事件，事件触发时执行done.apply(err,ee,event,args)
	// 返回值thunk(fn)用于重设回调函数done；thunk.cancel解绑事件、清空绑定函数
	function first(stuff, done) {
	  if (!Array.isArray(stuff))
	    throw new TypeError('arg must be an array of [ee, events...] arrays')
	
	  var cleanups = []
	
	  for (var i = 0; i < stuff.length; i++) {
	    var arr = stuff[i]
	
	    if (!Array.isArray(arr) || arr.length < 2)
	      throw new TypeError('each array member must be [ee, events...]')
	
	    var ee = arr[0]
	
	    for (var j = 1; j < arr.length; j++) {
	      var event = arr[j]
	      var fn = listener(event, callback)
	
	      // 监听event事件
	      ee.on(event, fn)
	
	      cleanups.push({
	        ee: ee,
	        event: event,
	        fn: fn,
	      })
	    }
	  }
	
	  // callback函数作为listener监听函数的回调，有事件触发时调用，并执行done.apply(err,ee,event,args)
	  function callback() {
	    cleanup()
	    done.apply(null, arguments)
	  }
	
	  // event事件触发或调用thunk.cancel方法时，解绑所有事件及清除绑定函数
	  function cleanup() {
	    var x
	    for (var i = 0; i < cleanups.length; i++) {
	      x = cleanups[i]
	      x.ee.removeListener(x.event, x.fn)
	    }
	  }
	
	  // 重设回调函数
	  function thunk(fn) {
	    done = fn
	  }
	
	  thunk.cancel = cleanup
	
	  return thunk
	}
	
	// event事件触发时，执行回调done(err,ee,event,args)
	// 参数err为错误对象或null，ee为事件对象，event为事件类型，args为参数
	function listener(event, done) {
	  return function onevent(arg1) {
	    var args = new Array(arguments.length)
	    var ee = this
	    var err = event === 'error'
	      ? arg1
	      : null
	
	    // copy args to prevent arguments escaping scope
	    for (var i = 0; i < args.length; i++) {
	      args[i] = arguments[i]
	    }
	
	    done(err, ee, event, args)
	  }
	}
