# maxBy

maxBy(iterator,transform,compare)，基于minBy，设置compare，输出最大比较值对应的迭代值。compare函数默认为数值比较。

    'use strict';

    var minBy = require('./minBy');

    var compareNumber = function compareNumber(a, b) {
      return a - b;
    };

    function maxBy(as, f, compare) {
      compare = compare || compareNumber;

      return minBy(as, f, function (u, v) {
        return compare(v, u);
      });
    }

    module.exports = maxBy;