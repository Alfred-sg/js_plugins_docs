# forEachAccumulated::react-dom

orEachAccumulated(arr,cb,scope)，遍历arr，以上下文scope执行cb函数。

    'use strict';
    
    // 首参为数组时，遍历数组项执行回调cb，传参为arr中每一项；首参非数组时，以scope为上下文执行cb，传参为arr
    function forEachAccumulated(arr, cb, scope) {
      if (Array.isArray(arr)) {
        arr.forEach(cb, scope);
      } else if (arr) {
        cb.call(scope, arr);
      }
    }
    
    module.exports = forEachAccumulated;