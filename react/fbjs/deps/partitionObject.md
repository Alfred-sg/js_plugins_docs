# partitionObject

partitionObject(object,predicate,context)，遍历object对象的属性，以context作为上下文执行predicate函数；将匹配与不匹配predicate回调的object属性分为两组，构成数组形式后输出。

    'use strict';

    var forEachObject = require('./forEachObject');

    // 将匹配与不匹配callback回调的object属性分为两组
    function partitionObject(object, callback, context) {
      var first = {};
      var second = {};
      forEachObject(object, function (value, key) {
        if (callback.call(context, value, key, object)) {
          first[key] = value;
        } else {
          second[key] = value;
        }
      });
      return [first, second];
    }

    module.exports = partitionObject;