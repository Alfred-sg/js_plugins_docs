# flatMapArray

flatMapArray(arr,fn)，遍历数组arr，执行回调fn.call(arr,arr[ii],ii)，将回调fn的返回值拼接为数组后输出。

    'use strict';

    var push = Array.prototype.push;

    /**
     * 遍历数组array，执行回调fn.call(array,array[ii],ii)，将回调fn的返回值拼接为数组后输出
     * @param  {array}    array 待遍历的数组
     * @param  {Function} fn    遍历执行函数，返回数组或null有效
     * @return {array}          由fn返回值子数组拼接的新数组
     */
    function flatMapArray(array, fn) {
      var ret = [];
      for (var ii = 0; ii < array.length; ii++) {
        var result = fn.call(array, array[ii], ii);
        if (Array.isArray(result)) {
          push.apply(ret, result);
        } else if (result != null) {
          throw new TypeError('flatMapArray: Callback must return an array or null, ' + 'received "' 
            + result + '" instead');
        }
      }
      return ret;
    }

    module.exports = flatMapArray;