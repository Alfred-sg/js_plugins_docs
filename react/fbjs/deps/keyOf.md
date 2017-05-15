# keyOf

keyOf(oneKeyObj)，返回单属性对象oneKeyObj的属性，或null。

    "use strict";

    // 返回单属性对象oneKeyObj的属性，或null
    var keyOf = function keyOf(oneKeyObj) {
      var key;
      for (key in oneKeyObj) {
        if (!oneKeyObj.hasOwnProperty(key)) {
          continue;
        }
        return key;
      }
      return null;
    };

    module.exports = keyOf;