# http-assert 1.2.0

## 概述

http-assert当某条件不符时，通过http-errors模块抛出{message:msg, status, statusCode, expose, ...opts}错误，作为koa中间件函数上下文对象context的assert方法，可通过context的onerrer处理错误对象。

## 源码

	var createError = require('http-errors');
	var eql = require('deep-equal');
	
	module.exports = assert;
	
	// value为否时，抛出{message:msg,status,statusCode,expose,...opts}错误
	function assert(value, status, msg, opts) {
	  if (value) return;
	  throw createError(status, msg, opts);
	}
	
	// a==b为否时，抛出{message:msg,status,statusCode,expose,...opts}错误
	assert.equal = function(a, b, status, msg, opts) {
	  assert(a == b, status, msg, opts);
	};
	
	// a!=b为否时，抛出{message:msg,status,statusCode,expose,...opts}错误
	assert.notEqual = function(a, b, status, msg, opts) {
	  assert(a != b, status, msg, opts);
	};
	
	// a===b为否时，抛出{message:msg,status,statusCode,expose,...opts}错误
	assert.strictEqual = function(a, b, status, msg, opts) {
	  assert(a === b, status, msg, opts);
	};
	
	// a!==b为否时，抛出{message:msg,status,statusCode,expose,...opts}错误
	assert.notStrictEqual = function(a, b, status, msg, opts) {
	  assert(a !== b, status, msg, opts);
	};
	
	// a、b对象不完全相等时，抛出{message:msg,status,statusCode,expose,...opts}错误
	assert.deepEqual = function(a, b, status, msg, opts) {
	  assert(eql(a, b), status, msg, opts);
	};
	
	// a、b对象完全相等时，抛出{message:msg,status,statusCode,expose,...opts}错误
	assert.notDeepEqual = function(a, b, status, msg, opts) {
	  assert(!eql(a, b), status, msg, opts);
	};
