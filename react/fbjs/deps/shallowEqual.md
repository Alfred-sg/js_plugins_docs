# shallowEqual

shallowEqual(a,b)，当a、b为普通数据类型时，比较值相等；为数组或对象，比较数组长度、属性个数是否相等，通过"==="浅比较首层数组项是否相等。

    'use strict';

    var hasOwnProperty = Object.prototype.hasOwnProperty;

    // 比较值相等
    function is(x, y) {
      if (x === y) {
        // +0 != -0 特殊情况
        return x !== 0 || y !== 0 || 1 / x === 1 / y;
      } else {
        // NaN == NaN 特殊情况
        return x !== x && y !== y;
      }
    }

    // 比较值相等，及浅比较键值相等
    function shallowEqual(objA, objB) {
      if (is(objA, objB)) {
        return true;
      }

      if (typeof objA !== 'object' || objA === null || typeof objB !== 'object' || objB === null) {
        return false;
      }

      var keysA = Object.keys(objA);// Object.keys也可作用于数组
      var keysB = Object.keys(objB);

      if (keysA.length !== keysB.length) {
        return false;
      }

      for (var i = 0; i < keysA.length; i++) {
        if (!hasOwnProperty.call(objB, keysA[i]) || !is(objA[keysA[i]], objB[keysA[i]])) {
          return false;
        }
      }

      return true;
    }

    module.exports = shallowEqual;