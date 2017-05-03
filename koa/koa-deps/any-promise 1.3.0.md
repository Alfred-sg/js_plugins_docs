# any-promise 1.3.0

## 概述和使用

any-promise模块用于输出Promise对象。当global.Promise存在时，以global.Promise输出；若不存在，以["es6-promise", "promise","native-promise-only","bluebird","rsvp","when", "q","pinkie","lie", "vow"]的顺序尝试加载依赖中已有的Promise并输出；否则报错。

	require("any-promise");// 输出global.Promise或"es6-promise"等模块的Promise
	
	var register = require("any-promise/register");
	
	register('q', {Promise: require('q').Promise}).Promise// 以次参的Promise为Promise属性输出
	register('global.Promise').Promise// 输出global.Promise
	register('q').Promise// 输出require('q').Promise
	register('xxx').Promise// global.Promise未定义，尝试以以require("es6-promise").Promise等输出，或报错

## 核心源码

### loader.js函数式执行Promise对象的注入策略

	"use strict"
	var REGISTRATION_KEY = '@@any-promise/REGISTRATION',
	    registered = null
	
	// 参数root即根对象，node端的global、浏览器端的window
	// 参数loadImplementation以函数形式约定Promise对象的加载策略，opts.Promise未定义时启用
	// 返回函数设定Promise对象，并输出{Promise,implementation}
	// 通常为内部使用，如register.js模块中调用，外部调用须设定loadImplementation参数加载策略
	module.exports = function(root, loadImplementation){
	  return function register(implementation, opts){
	    implementation = implementation || null
	    opts = opts || {}
	
	    // opts.global非false时，"any-promise"插件加载的Promise对象赋值给root[REGISTRATION_KEY]，不允许重复加载
	    // 再次调用时将root[REGISTRATION_KEY]作为"any-promise"插件的Promise对象
	    var registerGlobal = opts.global !== false;
	    if(registered === null && registerGlobal){
	      registered = root[REGISTRATION_KEY] || null
	    }
	
	    // "any-promise"插件已加载Promise对象，重复加载予以报错
	    if(registered !== null
	        && implementation !== null
	        && registered.implementation !== implementation){
	      throw new Error('any-promise already defined as "'+registered.implementation+
	        '".  You can only register an implementation before the first '+
	        ' call to require("any-promise") and an implementation cannot be changed')
	    }
	
	    if(registered === null){
	      // opts.Promise已定义，注入为"any-promise"插件加载的Promise对象
	      if(implementation !== null && typeof opts.Promise !== 'undefined'){
	        registered = {
	          Promise: opts.Promise,
	          implementation: implementation
	        }
	
	      // opts.Promise未定义，"any-promise"的Promise对象赋值为global.Promise对象；或implementation插件
	      //    或"es6-promise"等默认加载插件(需要依赖中含有该插件)；或者报错
	      } else {
	        registered = loadImplementation(implementation)
	      }
	
	      if(registerGlobal){
	        // root[REGISTRATION_KEY]用以判断是否重复加载，opts.global非false时开启判断
	        root[REGISTRATION_KEY] = registered
	      }
	    }
	
	    return registered
	  }
	}

### register.js注入Promise对象的策略

	"use strict"
	// 返回loader.js模块的返回值register(implementation,opts)函数
	// register('q', {Promise: require('q').Promise}) "any-promise"插件以opts.Promise为Promise对象
	// register('global.Promise') "any-promise"插件以global.Promise为Promise对象
	// register('q') "any-promise"插件以require(implementation).Promise为Promise对象
	// register('xxx') "any-promise"插件以require("es6-promise").Promise为Promise对象
	//    若"es6-promise"依赖存在；不存在，尝试以require("promise").Promise为Promise对象；默认Promise依赖均不存在，报错
	module.exports = require('./loader')(global, loadImplementation);
	
	// "any-promise"插件加载Promise对象的策略
	// implementation为'global.Promise'，加载global.Promise对象；或者node版本中含有Promise，加载该Promise对象
	// implementation指代promise模块名，加载"es6-promise"等模块，作为"any-promise"插件输出的Promise对象
	// 否则自动以["es6-promise", "promise","native-promise-only","bluebird","rsvp","when", "q","pinkie",
	//    "lie", "vow"]顺序加载Promise对象
	function loadImplementation(implementation){
	  var impl = null
	
	  // 加载global.Promise对象作为"any-promise"插件输出的Promise对象
	  // impl.implementation可视为"any-promise"插件输出Promise对象的类型标识符
	  if(shouldPreferGlobalPromise(implementation)){
	    impl = {
	      Promise: global.Promise,
	      implementation: 'global.Promise'
	    }
	
	  // 自定义加载何种Promise对象
	  } else if(implementation){
	    var lib = require(implementation)
	    impl = {
	      Promise: lib.Promise || lib,
	      implementation: implementation
	    }
	
	  // ["es6-promise", "promise","native-promise-only","bluebird","rsvp","when", "q","pinkie","lie", "vow"]
	  // 默认加载的Promise对象
	  } else {
	    impl = tryAutoDetect()
	  }
	
	  if(impl === null){
	    throw new Error('Cannot find any-promise implementation nor'+
	      ' global.Promise. You must install polyfill or call'+
	      ' require("any-promise/register") with your preferred'+
	      ' implementation, e.g. require("any-promise/register/bluebird")'+
	      ' on application load prior to any require("any-promise").')
	  }
	
	  return impl
	}
	
	// 强制加载global.Promise
	function shouldPreferGlobalPromise(implementation){
	  if(implementation){
	    return implementation === 'global.Promise'
	  } else if(typeof global.Promise !== 'undefined'){
	    // Load global promise if implementation not specified
	    // Versions < 0.11 did not have global Promise
	    // Do not use for version < 0.12 as version 0.11 contained buggy versions
	    var version = (/v(\d+)\.(\d+)\.(\d+)/).exec(process.version)
	    return !(version && +version[1] == 0 && +version[2] < 12)
	  }
	
	  return false
	}
	
	// 默认加载的Promise对象，需要依赖中含有"es6-promise"等插件
	function tryAutoDetect(){
	  var libs = [
	      "es6-promise",
	      "promise",
	      "native-promise-only",
	      "bluebird",
	      "rsvp",
	      "when",
	      "q",
	      "pinkie",
	      "lie",
	      "vow"]
	  var i = 0, len = libs.length
	  for(; i < len; i++){
	    try {
	      return loadImplementation(libs[i])
	    } catch(e){}
	  }
	  return null
	}

## 对外接口

### index.js 

node端加载global.Promise，或"es6-promise"等，或报错

	module.exports = require('./register')().Promise
	
### optional.js 

node端加载global.Promise，或"es6-promise"等，捕获错误

	"use strict";
	try {
	  module.exports = require('./register')().Promise || null
	} catch(e) {
	  module.exports = null
}

### register-shim.js 

浏览器端加载window.Promise，或报错

	"use strict";
	module.exports = require('./loader')(window, loadImplementation)
	
	function loadImplementation(){
	  if(typeof window.Promise === 'undefined'){
	    throw new Error("any-promise browser requires a polyfill or explicit registration"+
	      " e.g: require('any-promise/register/bluebird')")
	  }
	  return {
	    Promise: window.Promise,
	    implementation: 'window.Promise'
	  }
	}
	
### implementation.js

输出加载Promise对象的标识，如"global.Promise"、"es6-promise"等字符串值

	module.exports = require('./register')().implementation
	
### register/*.js

以插件设定Promise对象

1. bluebird.js: require('../register')('bluebird', {Promise: require('bluebird')})
2. es6-promise.js: require('../register')('es6-promise', {Promise: require('es6-promise').Promise})
3. lie.js: require('../register')('lie', {Promise: require('lie')})
4. native-promise-only.js: require('../register')('native-promise-only', {Promise: require('native-promise-only')})
5. pinkie.js: require('../register')('pinkie', {Promise: require('pinkie')})
6. promise.js: require('../register')('promise', {Promise: require('promise')})
7. q.js: require('../register')('q', {Promise: require('q').Promise})
8. rsvp.js: require('../register')('rsvp', {Promise: require('rsvp').Promise})
9. vow.js: require('../register')('vow', {Promise: require('vow').Promise})
10. when.js: require('../register')('when', {Promise: require('when').Promise})


