# koa-convert 1.2.0

koa-convert模块用于将旧版本生成器形式的中间件函数function * (next){}转化为类async函数function(ctx,next){}，前者无返回值，后者返回promise。

模块中同时包含componse方法纯用生成器函数构造中间件，back方法将转化后的类async函数回滚到生成器函数。

	'use strict'
	
	const co = require('co')// 生成器函数中异步逻辑处理，将异步返回值作为生成器next方法的参数
	const compose = require('koa-compose')// 将koa的中间件函数串联为一
	
	module.exports = convert
	
	// 参数mw即koa().use(function * generator(next){})挂载的中间件函数，生成器generator
	function convert (mw) {
	  if (typeof mw !== 'function') {
	    throw new TypeError('middleware must be a function')
	  }
	  if (mw.constructor.name !== 'GeneratorFunction') {
	    // assume it's Promise-based middleware
	    return mw
	  }
	
	  // 将下一个中间件调用函数next转化为生成器函数，同时为mw注入koa构建的上下文对象ctx及生成器化的next函数
	  // 使用co串联，意义是处理mw生成器函数中调用多个生成器函数实现异步逻辑
	  const converted = function (ctx, next) {
	    return co.call(ctx, mw.call(ctx, createGenerator(next)))
	  }
	
	  converted._name = mw._name || mw.name
	  return converted
	}
	
	function * createGenerator (next) {
	  return yield next()
	}
	
	// 处理多个生成器函数，转化为koa最终在callback使用的串联异步逻辑的函数function(ctx)
	// 重构koa/application模块的callback方法时使用，
	// convert.compose(mw, mw, mw)
	// convert.compose([mw, mw, mw])
	convert.compose = function (arr) {
	  if (!Array.isArray(arr)) {
	    arr = Array.from(arguments)
	  }
	  return compose(arr.map(convert))
	}
	
	// 将async函数回滚成生成器函数function *(next){}，参数next也须是生成器函数
	convert.back = function (mw) {
	  if (typeof mw !== 'function') {
	    throw new TypeError('middleware must be a function')
	  }
	  if (mw.constructor.name === 'GeneratorFunction') {
	    // assume it's generator middleware
	    return mw
	  }
	  const converted = function * (next) {
	    let ctx = this
	    let called = false
	    yield Promise.resolve(mw(ctx, function () {
	      if (called) {
	        // guard against multiple next() calls
	        // https://github.com/koajs/compose/blob/4e3e96baf58b817d71bd44a8c0d78bb42623aa95/index.js#L36
	        return Promise.reject(new Error('next() called multiple times'))
	      }
	      called = true
	      return co.call(ctx, next)
	    }))
	  }
	  converted._name = mw._name || mw.name
	  return converted
	}