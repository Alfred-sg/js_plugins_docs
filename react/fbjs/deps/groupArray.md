# groupArray

groupArray(array,fn)，遍历array，执行回调fn.call(array,array[ii],ii)，以获取分组的属性名，将array打包分组。

    'use strict';

    /**
     * 遍历array，执行回调fn.call(array,array[ii],ii)，以获取分组的属性名，将array打包分组
     * @param  {array}   array  待分组的数组
     * @param  {Function} fn    遍历array以获取分组的属性名
     * @return {object}         分组后以对象形式输出
     */
    function groupArray(array, fn) {
      var ret = {};
      for (var ii = 0; ii < array.length; ii++) {
        var result = fn.call(array, array[ii], ii);
        if (!ret[result]) {
          ret[result] = [];
        }
        ret[result].push(array[ii]);
      }
      return ret;
    }

    module.exports = groupArray;