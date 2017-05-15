# nullthrows

nullthrows(arg)，首参arg不为null或undefined时，直接抛出；否则报"Got unexpected null or undefined"错误。

    "use strict";

    // 首参arg不为null或undefined时，直接抛出；否则报"Got unexpected null or undefined"错误
    var nullthrows = function nullthrows(x) {
      if (x != null) {
        return x;
      }
      throw new Error("Got unexpected null or undefined");
    };

    module.exports = nullthrows;