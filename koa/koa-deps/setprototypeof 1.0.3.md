# setprototypeof 1.0.3

## 概述

setprototypeof模块用于设置原型，或浅拷贝。

## 源码

	module.exports = Object.setPrototypeOf || ({__proto__:[]} instanceof Array ? setProtoOf : mixinProperties);
	
	function setProtoOf(obj, proto) {
		obj.__proto__ = proto;
		return obj;
	}
	
	function mixinProperties(obj, proto) {
		for (var prop in proto) {
			if (!obj.hasOwnProperty(prop)) {
				obj[prop] = proto[prop];
			}
		}
		return obj;
	}