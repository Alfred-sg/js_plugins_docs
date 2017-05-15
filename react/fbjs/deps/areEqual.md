# areEqual

areEqual(a,b)，校验a、b是否完全值相等。

	'use strict';

	var aStackPool = [];
	var bStackPool = [];

	// 完全相等比较
	/**
	 * 完全相等比较
	 * 
	 * @param  {[any]} 待比较值a，任何属性
	 * @param  {[any]} 待比较值b，任何属性
	 * @return {[boolean]} a、b完全相等，返回true；否则返回false
	 */
	function areEqual(a, b) {
	  // aStackPool、bStackPool有值时取出，通常经过一次比较后aStackPool、bStackPool存入单个空数组元素；无值时取空数组
	  var aStack = aStackPool.length ? aStackPool.pop() : [];
	  var bStack = bStackPool.length ? bStackPool.pop() : [];

	  var result = eq(a, b, aStack, bStack);

	  // 比较完成后清空aStack和bStack，仍然保持为数组形式
	  aStack.length = 0;
	  bStack.length = 0;

	  // aStackPool、bStackPool存入单个空数组元素，下次比较时取出，aStackPool、bStackPool长度不会递增
	  aStackPool.push(aStack);
	  bStackPool.push(bStack);

	  return result;
	}

	function eq(a, b, aStack, bStack) {
	  if (a === b) {
	    // 针对某些平台`+0 !== -0`的情况
	    return a !== 0 || 1 / a == 1 / b;
	  }
	  if (a == null || b == null) {
	    // 其中一个为null或undefined时返回false
	    return false;
	  }
	  if (typeof a != 'object' || typeof b != 'object') {
	    // 排除基本类型后，其中一个不是引用类型，返回false
	    return false;
	  }
	  var objToStr = Object.prototype.toString;
	  var className = objToStr.call(a);
	  if (className != objToStr.call(b)) {
	    // 类型判断
	    return false;
	  }

	  // 字符串、数值、日期对象、布尔类型、正则对象比较
	  switch (className) {
	    case '[object String]':
	      return a == String(b);
	    case '[object Number]':
	      return isNaN(a) || isNaN(b) ? false : a == Number(b);
	    case '[object Date]':
	    case '[object Boolean]':
	      return +a == +b;
	    case '[object RegExp]':
	      return a.source == b.source && a.global == b.global && a.multiline == b.multiline && a.ignoreCase == b.ignoreCase;
	  }

	  // 何种循环结构下触发？？？
	  var length = aStack.length;
	  while (length--) {
	    if (aStack[length] == a) {
	      return bStack[length] == b;
	    }
	  }

	  // aStack、bStack将存入a、b及其子属性、子数组项，两者相等时删除该比较项
	  aStack.push(a);
	  bStack.push(b);

	  var size = 0;

	  // 数组类型比较
	  if (className === '[object Array]') {
	    size = a.length;

	    // 长度不等
	    if (size !== b.length) {
	      return false;
	    }
	    // 深度比较数组项
	    while (size--) {
	      if (!eq(a[size], b[size], aStack, bStack)) {
	        return false;
	      }
	    }
	  } else {
	    // 对象的构造函数不等
	    if (a.constructor !== b.constructor) {
	      return false;
	    }

	    // 通过对象的valueOf方法返回原始值比较
	    if (a.hasOwnProperty('valueOf') && b.hasOwnProperty('valueOf')) {
	      return a.valueOf() == b.valueOf();
	    }

	    // 对象属性个数是否匹配
	    var keys = Object.keys(a);
	    if (keys.length != Object.keys(b).length) {
	      return false;
	    }

	    // 深度比较对象属性
	    for (var i = 0; i < keys.length; i++) {
	      if (!eq(a[keys[i]], b[keys[i]], aStack, bStack)) {
	        return false;
	      }
	    }
	  }
	  aStack.pop();
	  bStack.pop();
	  return true;
	}

	module.exports = areEqual;