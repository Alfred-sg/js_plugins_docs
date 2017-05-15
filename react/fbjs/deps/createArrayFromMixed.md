# createArrayFromMixed

createArrayFromMixed(obj|arr|nodeList)，将单个对象、数组或伪数组转化成数组后输出。

    'use strict';

    var invariant = require('./invariant');

    // 拆分为数组，obj不能是arguments
    function toArray(obj) {
      var length = obj.length;

      !(!Array.isArray(obj) && (typeof obj === 'object' || typeof obj === 'function')) ? 
        process.env.NODE_ENV !== 'production' ? invariant(false, 'toArray: Array-like object expected') 
        : invariant(false) : void 0;

      !(typeof length === 'number') ? process.env.NODE_ENV !== 'production' ? 
        invariant(false, 'toArray: Object needs a length property') : invariant(false) : void 0;

      !(length === 0 || length - 1 in obj) ? process.env.NODE_ENV !== 'production' ? 
        invariant(false, 'toArray: Object should have keys for indices') : invariant(false) : void 0;

      !(typeof obj.callee !== 'function') ? process.env.NODE_ENV !== 'production' ? 
        invariant(false, 'toArray: Object can\'t be `arguments`. Use rest params ' 
          + '(function(...args) {}) or Array.from() instead.') : invariant(false) : void 0;

      if (obj.hasOwnProperty) {
        try {
          return Array.prototype.slice.call(obj);
        } catch (e) {
          // IE < 9 does not support Array#slice on collections objects
        }
      }

      var ret = Array(length);
      for (var ii = 0; ii < length; ii++) {
        ret[ii] = obj[ii];
      }
      return ret;
    }

    // 是否数组或伪数组对象arguments、HTMLCollection/NodeList，排除null、false、window、node、
    function hasArrayNature(obj) {
      return (
        !!obj && (
        // NodeLists are functions in Safari
        typeof obj == 'object' || typeof obj == 'function') &&
        'length' in obj &&
        !('setInterval' in obj) &&
        typeof obj.nodeType != 'number' && (
        Array.isArray(obj) ||
        // arguments
        'callee' in obj ||
        // HTMLCollection/NodeList
        'item' in obj)
      );
    }

    /**
     * 转化成数组后输出
     * @param  {object|array|nodeList} 待转换的单对象、数组或伪数组
     * @return {array}     转换成数组后输出
     */
    function createArrayFromMixed(obj) {
      if (!hasArrayNature(obj)) {
        return [obj];
      } else if (Array.isArray(obj)) {
        return obj.slice();
      } else {
        return toArray(obj);
      }
    }

    module.exports = createArrayFromMixed;