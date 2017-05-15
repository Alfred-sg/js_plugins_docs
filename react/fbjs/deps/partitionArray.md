# partitionArray

partitionArray(array,predicate,context)，遍历array，执行回调predicate.call(context,array[ii],ii,array)；将数组array中匹配与不匹配predicate回调的元素分成两组后以数组形式返回。

    "use strict";

    // 将数组array中匹配与不匹配predicate回调的元素分成两组后以数组形式返回
    function partitionArray(array, predicate, context) {
      var first = [];
      var second = [];
      array.forEach(function (element, index) {
        if (predicate.call(context, element, index, array)) {
          first.push(element);
        } else {
          second.push(element);
        }
      });
      return [first, second];
    }

    module.exports = partitionArray;