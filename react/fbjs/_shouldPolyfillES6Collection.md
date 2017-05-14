# _shouldPolyfillES6Collection

shouldPolyfillES6Collection("Map|Set")，校验浏览器平台是否需要通过babel等转译后，才能支持Map、Set类型，及该类型的next迭代、keys、forEach、clear等方法。

	'use strict';

	/**
	 * 校验浏览器平台是否需要通过babel等转译后，才能支持Map、Set类型，及该类型的next迭代、keys、forEach、clear等方法
	 * 
	 * @param  {[string]} 字符串"Map"或"Set"
	 * @return {[boolean]} 正值需要babel等转译工具转译，否值浏览器原生支持
	 */
	function shouldPolyfillES6Collection(collectionName) {
	  var Collection = global[collectionName];

	  // 全局对象下无Map、Set属性
	  if (Collection == null) {
	    return true;
	  }

	  // Map、Set内建的迭代器需要global.Symbol语法支持
	  if (typeof global.Symbol !== 'function') {
	    return true;
	  }

	  var proto = Collection.prototype;

	  // Collection不是构造函数，或实例没有clear、keys、forEach方法或size属性，需要babel等转译工具转译
	  return Collection == null || typeof Collection !== 'function' || 
	    typeof proto.clear !== 'function' || new Collection().size !== 0 || 
	    typeof proto.keys !== 'function' || typeof proto.forEach !== 'function';
	}

	module.exports = shouldPolyfillES6Collection;