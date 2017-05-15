# removeFromArray

removeFromArray(array,element)，从数组array中移除元素element。

    "use strict";

    // 从数组array中移除元素element
    function removeFromArray(array, element) {
      var index = array.indexOf(element);
      if (index !== -1) {
        array.splice(index, 1);
      }
    }

    module.exports = removeFromArray;