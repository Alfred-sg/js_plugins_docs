# hyphenate

hyphenate(string)，将参数string从小驼峰式书写形式转化为连字符书写。

    'use strict';

    var _uppercasePattern = /([A-Z])/g;

    /**
     * 将参数string从小驼峰式书写形式转化为连字符书写
     * @param  {string} string 待转化的字符串
     * @return {string}        转化为连字符书写
     */
    function hyphenate(string) {
      return string.replace(_uppercasePattern, '-$1').toLowerCase();
    }

    module.exports = hyphenate;