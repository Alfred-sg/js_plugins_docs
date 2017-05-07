# only 0.0.2

## 概述

only模块根据白名单获取obj对象的属性。

## 源码

	module.exports = function(obj, keys){
	  obj = obj || {};
	  if ('string' == typeof keys) keys = keys.split(/ +/);
	  return keys.reduce(function(ret, key){
	    if (null == obj[key]) return ret;
	    ret[key] = obj[key];
	    return ret;
	  }, {});
	};