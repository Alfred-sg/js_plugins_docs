# equalsSet

equalsSet(a,b)，比较两个set实例a、b是否等值。

    'use strict';

    var everySet = require('./everySet');

    // 比较两个set实例是否等值
    function equalsSet(one, two) {
      if (one.size !== two.size) {
        return false;
      }
      return everySet(one, function (value) {
        return two.has(value);
      });
    }

    module.exports = equalsSet;