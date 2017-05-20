# someSet

someSet(set,predicate,context)，遍历set，当有成员匹配predicate.bind(context)时，返回真值；否则返回否值。

    'use strict';

    // 遍历set，当有成员匹配callback.bind(context)时，返回真值；否则返回否值
    function someSet(set, callback, context) {
      var iterator = set.entries();
      var current = iterator.next();
      while (!current.done) {
        var entry = current.value;
        if (callback.call(context, entry[1], entry[0], set)) {
          return true;
        }
        current = iterator.next();
      }
      return false;
    }

    module.exports = someSet;