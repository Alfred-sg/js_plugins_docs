# camelize

camelize(str)，将字符串从连字符书写形式转化为小驼峰式书写形式。

    "use strict";

    var _hyphenPattern = /-(.)/g;

    /**
     * 将参数string从连字符书写形式转化为小驼峰式书写
     * @param  {string} string 待转化的字符串
     * @return {string}        转化为小驼峰式字符串
     */
    function camelize(string) {
      return string.replace(_hyphenPattern, function (_, character) {
        return character.toUpperCase();
      });
    }

    module.exports = camelize;