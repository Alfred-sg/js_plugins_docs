# sprintf

sprintf(format)，以次参等替换首参format字符串中的%s后输出。

    "use strict";

    // 以次参等替换首参format字符串中的%s后输出
    function sprintf(format) {
      for (var _len = arguments.length, args = Array(_len > 1 ? _len - 1 : 0), _key = 1; _key < _len; _key++) {
        args[_key - 1] = arguments[_key];
      }

      var index = 0;
      return format.replace(/%s/g, function (match) {
        return args[index++];
      });
    }

    module.exports = sprintf;