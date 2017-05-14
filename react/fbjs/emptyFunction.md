# emptyFunction

各类空函数。

    "use strict";

    function makeEmptyFunction(arg) {
      return function () {
        return arg;
      };
    }

    // 返回undefined
    var emptyFunction = function emptyFunction() {};

    // 返回将参数作为返回值的函数
    emptyFunction.thatReturns = makeEmptyFunction;

    // 返回false
    emptyFunction.thatReturnsFalse = makeEmptyFunction(false);

    // 返回true
    emptyFunction.thatReturnsTrue = makeEmptyFunction(true);

    // 返回null
    emptyFunction.thatReturnsNull = makeEmptyFunction(null);

    // 返回this
    emptyFunction.thatReturnsThis = function () {
      return this;
    };

    // 参数原路返回
    emptyFunction.thatReturnsArgument = function (arg) {
      return arg;
    };

    module.exports = emptyFunction;