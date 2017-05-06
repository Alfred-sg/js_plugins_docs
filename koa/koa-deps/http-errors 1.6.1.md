# http-errors 1.6.1

## 概述及使用

http-errors用于构建http error错误对象{message, status, statusCode, expose, ...props}。使用koa中间件上下文对象context的onError方法处理该错误时，400客户端错误将作为响应内容传给客户端；500服务器端错误因err.expose=false将予以报错处理。

	var createError = require('http-errors')
	var express = require('express')
	var app = express()
	
	app.use(function (req, res, next) {
	  if (!req.user) return next(createError(401, 'Please login to view this page.'))
	  next()
	})

## 源码

	'use strict'
	
	var deprecate = require('depd')('http-errors')
	var setPrototypeOf = require('setprototypeof')
	var statuses = require('statuses')
	var inherits = require('inherits')
	
	module.exports = createError
	module.exports.HttpError = createHttpErrorConstructor()
	
	populateConstructorExports(module.exports, statuses.codes, module.exports.HttpError)
	
	function codeClass (status) {
	  return Number(String(status).charAt(0) + '00')
	}
	
	// createError([err,msg,status,props])构建HTTP Error错误对象{status,statusCode,expose,...props}
	// 参数以类型区别对象，与顺序无关
	// statusCode为4**或5**，默认取500
	function createError () {
	  var err
	  var msg
	  var status = 500
	  var props = {}
	  for (var i = 0; i < arguments.length; i++) {
	    var arg = arguments[i]
	    if (arg instanceof Error) {
	      err = arg
	      status = err.status || err.statusCode || status
	      continue
	    }
	    switch (typeof arg) {
	      case 'string':
	        msg = arg
	        break
	      case 'number':
	        status = arg
	        if (i !== 0) {
	          deprecate('non-first-argument status code; replace with createError(' + arg + ', ...)')
	        }
	        break
	      case 'object':
	        props = arg
	        break
	    }
	  }
	
	  if (typeof status === 'number' && (status < 400 || status >= 600)) {
	    deprecate('non-error status code; use only 4xx or 5xx status codes')
	  }
	
	  if (typeof status !== 'number' ||
	    (!statuses[status] && (status < 400 || status >= 600))) {
	    status = 500
	  }
	
	  // createError存储状态码到错误对象的映射
	  var HttpError = createError[status] || createError[codeClass(status)]
	
	  if (!err) {
	    err = HttpError
	      ? new HttpError(msg)
	      : new Error(msg || statuses[status])
	    Error.captureStackTrace(err, createError)
	  }
	
	  if (!HttpError || !(err instanceof HttpError) || err.status !== status) {
	    err.expose = status < 500
	    err.status = err.statusCode = status
	  }
	
	  for (var key in props) {
	    if (key !== 'status' && key !== 'statusCode') {
	      err[key] = props[key]
	    }
	  }
	
	  return err
	}
	
	// HttpError作为基类，抽象类，不能直接调用
	function createHttpErrorConstructor () {
	  function HttpError () {
	    throw new TypeError('cannot construct abstract class')
	  }
	
	  inherits(HttpError, Error)
	
	  return HttpError
	}
	
	// 4**状态码构建客户端错误，参数HttpError为createHttpErrorConstructor构造的错误对象基类
	// name为大驼峰式状态码文案，code状态码
	function createClientErrorConstructor (HttpError, name, code) {
	  var className = name.match(/Error$/) ? name : name + 'Error'
	
	  function ClientError (message) {
	    var msg = message != null ? message : statuses[code]
	    var err = new Error(msg)
	
	    Error.captureStackTrace(err, ClientError)
	
	    setPrototypeOf(err, ClientError.prototype)
	
	    Object.defineProperty(err, 'message', {
	      enumerable: true,
	      configurable: true,
	      value: msg,
	      writable: true
	    })
	
	    Object.defineProperty(err, 'name', {
	      enumerable: false,
	      configurable: true,
	      value: className,
	      writable: true
	    })
	
	    return err
	  }
	
	  inherits(ClientError, HttpError)
	
	  ClientError.prototype.status = code
	  ClientError.prototype.statusCode = code
	  ClientError.prototype.expose = true
	
	  return ClientError
	}
	
	// 5**状态码构建服务器端错误，参数HttpError为createHttpErrorConstructor构造的错误对象基类
	// name为大驼峰式状态码文案，code状态码
	function createServerErrorConstructor (HttpError, name, code) {
	  var className = name.match(/Error$/) ? name : name + 'Error'
	
	  function ServerError (message) {
	    var msg = message != null ? message : statuses[code]
	    var err = new Error(msg)
	
	    Error.captureStackTrace(err, ServerError)
	
	    setPrototypeOf(err, ServerError.prototype)
	
	    Object.defineProperty(err, 'message', {
	      enumerable: true,
	      configurable: true,
	      value: msg,
	      writable: true
	    })
	
	    Object.defineProperty(err, 'name', {
	      enumerable: false,
	      configurable: true,
	      value: className,
	      writable: true
	    })
	
	    return err
	  }
	
	  inherits(ServerError, HttpError)
	
	  ServerError.prototype.status = code
	  ServerError.prototype.statusCode = code
	  ServerError.prototype.expose = false
	
	  return ServerError
	}
	
	// 为exports注入400~599状态码或大驼峰式状态码文案到错误对象的映射
	function populateConstructorExports (exports, codes, HttpError) {
	  codes.forEach(function forEachCode (code) {
	    var CodeError
	    var name = toIdentifier(statuses[code])// 将状态码文案转化为大驼峰书写法
	
	    switch (codeClass(code)) {
	      // 4**状态码构建客户端错误{status,statusCode,expose:true}
	      case 400:
	        CodeError = createClientErrorConstructor(HttpError, name, code)
	        break
	      // 5**状态码构建服务器端错误{status,statusCode,expose:false}
	      case 500:
	        CodeError = createServerErrorConstructor(HttpError, name, code)
	        break
	    }
	
	    if (CodeError) {
	      exports[code] = CodeError
	      exports[name] = CodeError
	    }
	  })
	
	  // backwards-compatibility
	  exports["I'mateapot"] = deprecate.function(exports.ImATeapot,
	    '"I\'mateapot"; use "ImATeapot" instead')
	}
	
	// 将状态码文案转化为大驼峰书写法
	function toIdentifier (str) {
	  return str.split(' ').map(function (token) {
	    return token.slice(0, 1).toUpperCase() + token.slice(1)
	  }).join('').replace(/[^ _0-9a-z]/gi, '')
	}