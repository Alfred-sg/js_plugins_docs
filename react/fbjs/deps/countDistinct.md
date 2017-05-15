# countDistinct

countDistinct(iter,selector)，通过selector转换迭代器iter中各迭代值后，通过Set类型后计数。

    'use strict';

    var Set = require('./Set');

    var emptyFunction = require('./emptyFunction');

    /**
     * 通过selector转换迭代器iter中各迭代值后，通过Set类型后计数
     * @param  {iterator} iter     迭代器，next方法取值
     * @param  {fn} selector      用于转换迭代值，通过Set过滤重复值
     * @return {number}          对迭代值计算
     */
    function countDistinct(iter, selector) {
      selector = selector || emptyFunction.thatReturnsArgument;

      var set = new Set();
      var _iteratorNormalCompletion = true;
      var _didIteratorError = false;
      var _iteratorError = undefined;

      try {
        for (var _iterator = iter[Symbol.iterator](), _step; 
          !(_iteratorNormalCompletion = (_step = _iterator.next()).done); 
          _iteratorNormalCompletion = true) {
          var val = _step.value;

          set.add(selector(val));
        }
      } catch (err) {
        _didIteratorError = true;
        _iteratorError = err;
      } finally {
        try {
          if (!_iteratorNormalCompletion && _iterator['return']) {
            _iterator['return']();
          }
        } finally {
          if (_didIteratorError) {
            throw _iteratorError;
          }
        }
      }

      return set.size;
    }

    module.exports = countDistinct;