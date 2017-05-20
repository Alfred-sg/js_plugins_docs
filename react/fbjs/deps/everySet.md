# everySet

everySet(set,predicate,context)，遍历set，当所有成员均匹配predicate.bind(context)时，返回真值；否则返回否值。

    'use strict';

    // everySet(set,callback,context)，遍历set，当所有成员均匹配callback.bind(context)时，返回真值；否则返回否值
    function everySet(set, callback, context) {
      var iterator = set.entries();
      var current = iterator.next();
      while (!current.done) {
        var entry = current.value;
        if (!callback.call(context, entry[1], entry[0], set)) {
          return false;
        }
        current = iterator.next();
      }
      return true;
    }

    module.exports = everySet;