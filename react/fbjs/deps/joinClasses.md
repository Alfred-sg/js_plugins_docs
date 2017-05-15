# joinClasses

joinClasses(className)，将多个样式字符串拼接为单样式形式。

    'use strict';

    // 将多个样式字符串拼接为单样式形式
    function joinClasses(className /*, ... */) {
      if (!className) {
        className = '';
      }
      var nextClass = void 0;
      var argLength = arguments.length;
      if (argLength > 1) {
        for (var ii = 1; ii < argLength; ii++) {
          nextClass = arguments[ii];
          if (nextClass) {
            className = (className ? className + ' ' : '') + nextClass;
          }
        }
      }
      return className;
    }

    module.exports = joinClasses;