# minby

minBy(iterator,transform,compare)，遍历迭代器iterator，将迭代值传入transform转换函数中，以获取比较值，通过比较函数compare作比较，输出最小比较值对应的迭代值。

"use strict";

var compareNumber = function compareNumber(a, b) {
  return a - b;
};

/**
 * 遍历迭代器as，通过回调f传参迭代值获取比较值，以compare函数比较后，输出最小值对应的迭代值
 * @param  {iterator} as      迭代器，遍历获取迭代值
 * @param  {function} f       将迭代值转化为比较值
 * @param  {function} compare 比较函数
 * @return {[type]}           输出最小比较值所对应的迭代值
 */
function minBy(as, f, compare) {
  compare = compare || compareNumber;

  var minA = undefined;
  var minB = undefined;
  var seenFirst = false;
  var _iteratorNormalCompletion = true;
  var _didIteratorError = false;
  var _iteratorError = undefined;

  try {
    for (var _iterator = as[Symbol.iterator](), _step; !(_iteratorNormalCompletion = (_step = _iterator.next()).done); _iteratorNormalCompletion = true) {
      var a = _step.value;

      var b = f(a);
      if (!seenFirst || compare(b, minB) < 0) {
        minA = a;
        minB = b;
        seenFirst = true;
      }
    }
  } catch (err) {
    _didIteratorError = true;
    _iteratorError = err;
  } finally {
    try {
      if (!_iteratorNormalCompletion && _iterator["return"]) {
        _iterator["return"]();
      }
    } finally {
      if (_didIteratorError) {
        throw _iteratorError;
      }
    }
  }

  return minA;
}

module.exports = minBy;