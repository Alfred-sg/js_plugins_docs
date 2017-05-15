# flattenArray

flattenArray(arr)，将深度嵌套的数组arr转化为扁平化数组后输出。

    "use strict";

    /**
     * 将深度嵌套的数组转化为扁平化数组后输出
     * @param  {array} array 深度嵌套的数组
     * @return {array}       扁平化数组
     */
    function flattenArray(array) {
      var result = [];
      flatten(array, result);
      return result;
    }

    function flatten(array, result) {
      var length = array.length;
      var ii = 0;

      while (length--) {
        var current = array[ii++];
        if (Array.isArray(current)) {
          flatten(current, result);
        } else {
          result.push(current);
        }
      }
    }

    module.exports = flattenArray;