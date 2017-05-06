# delegates 1.0.0

## 概述及使用

delegates模块实现proto对象代理调用、访问或设置proto.target对象的方法或属性。

### koa中使用

使context中间件函数的上下文对象具有koa封装的请求request、响应response的方法和属性。

	var delegate = require('delegates');
	
	delegate(proto, 'request')
	  .method('acceptsLanguages')
	  .method('acceptsEncodings')
	  .method('acceptsCharsets')
	  .method('accepts')
	  .method('is')
	  .access('querystring')
	  .access('idempotent')
	  .access('socket')
	  .access('length')
	  .access('query')
	  .access('search')
	  .access('status')
	  .access('method')
	  .access('path')
	  .access('body')
	  .access('host')
	  .access('url')
	  .getter('subdomains')
	  .getter('protocol')
	  .getter('header')
	  .getter('stale')
	  .getter('fresh')
	  .getter('secure')
	  .getter('ips')
	  .getter('ip')
	  
## 源码

	module.exports = Delegator;
	
	// delege(proto,target).methos("push") 通过proto对象代理调用、访问或设置proto.target对象的方法或属性
	function Delegator(proto, target) {
	  if (!(this instanceof Delegator)) return new Delegator(proto, target);
	  this.proto = proto;
	  this.target = target;
	  this.methods = [];
	  this.getters = [];
	  this.setters = [];
	  this.fluents = [];
	}
	
	// 调用方法，proto[method]代理target[method]，且以target作为上下文
	Delegator.prototype.method = function(name){
	  var proto = this.proto;
	  var target = this.target;
	  this.methods.push(name);
	
	  proto[name] = function(){
	    // this指向proto对象
	    return this[target][name].apply(this[target], arguments);
	  };
	
	  return this;
	};
	
	// 既允许赋值又允许取值，proto[name]代理target[name]
	Delegator.prototype.access = function(name){
	  return this.getter(name).setter(name);
	};
	
	// 取值，proto[name]代理target[name]
	Delegator.prototype.getter = function(name){
	  var proto = this.proto;
	  var target = this.target;
	  this.getters.push(name);
	
	  proto.__defineGetter__(name, function(){
	    return this[target][name];
	  });
	
	  return this;
	};
	
	// 赋值，proto[name]代理target[name]
	Delegator.prototype.setter = function(name){
	  var proto = this.proto;
	  var target = this.target;
	  this.setters.push(name);
	
	  proto.__defineSetter__(name, function(val){
	    return this[target][name] = val;
	  });
	
	  return this;
	};
	
	// 根据传参的有无取值赋值，proto[name]代理target[name]
	Delegator.prototype.fluent = function (name) {
	  var proto = this.proto;
	  var target = this.target;
	  this.fluents.push(name);
	
	  proto[name] = function(val){
	    if ('undefined' != typeof val) {
	      this[target][name] = val;
	      return this;
	    } else {
	      return this[target][name];
	    }
	  };
	
	  return this;
	};