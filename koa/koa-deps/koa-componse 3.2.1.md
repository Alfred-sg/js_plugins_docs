# koa-componse 3.2.1

顺序异步执行多个中间件函数，返回promise对象，用以添加回调、错误处理。

	'use strict'
	
	const Promise = require('any-promise')// Promise延迟对象

	module.exports = compose
	
	// 以Promise对象顺序异步执行多个中间件函数，koa构造的context以引用对象形式被处理以判断请求、修改相应
	function compose (middleware) {
	  // 校验middleware为数组且数组项为函数
	  if (!Array.isArray(middleware)) throw new TypeError('Middleware stack must be an array!')
	  for (const fn of middleware) {
	    if (typeof fn !== 'function') throw new TypeError('Middleware must be composed of functions!')
	  }
	
	  // 参数context，koa构造的上下文对象
	  // 参数next，koa默认为空，赋值也无意义
	  return function (context, next) {
	    // last called middleware #
	    let index = -1
	    return dispatch(0)
	    function dispatch (i) {
	      if (i <= index) return Promise.reject(new Error('next() called multiple times'))
	      index = i
	      let fn = middleware[i]
	      if (i === middleware.length) fn = next
	      if (!fn) return Promise.resolve()// 最终Promise，不传参，以引用对象context获得最终响应
	      try {
	        // 通过Promise.resolve顺序异步执行多个中间件函数，传参为引用对象context和下一个中间件函数的执行器next函数
	        return Promise.resolve(fn(context, function next () {
	          return dispatch(i + 1)
	        }))
	      } catch (err) {
	        // 返回Promise通过catch方法捕获错误，并处理
	        return Promise.reject(err)
	      }
	    }
	  }
	}

