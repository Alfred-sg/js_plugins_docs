# concatAllArray

concatAllArray([arr1,...arrN])，将二维数组的各数组项拼接为单一数组后返回。

    'use strict';

    var push = Array.prototype.push;

    /**
     * 拼接二维数组的数组项为单一数组
     *
     * @param {array} array 二维数组
     * @return {array} 将二维数组的各数组项拼接为单一数组后返回
     */
    function concatAllArray(array) {
      var ret = [];
      for (var ii = 0; ii < array.length; ii++) {
        var value = array[ii];
        if (Array.isArray(value)) {
          push.apply(ret, value);
        } else if (value != null) {
          throw new TypeError('concatAllArray: All items in the array must be an array or null, ' 
            + 'got "' + value + '" at index "' + ii + '" instead');
        }
      }
      return ret;
    }

    module.exports = concatAllArray;