# emptyObject

emptyObject，创建冻结的空对象。	

	'use strict';

	var emptyObject = {};

	if (process.env.NODE_ENV !== 'production') {
	  Object.freeze(emptyObject);
	}

	// 空对象，并且用Object.freeze冻结为不可修改、不可扩展
	module.exports = emptyObject;