# hyphenateStyleName

hyphenateStyleName(style)，将小驼峰式书写的样式名转化为连字符式，特别将"ms-"起始的样式名转化成以"-ms-"起始。

    'use strict';

    var hyphenate = require('./hyphenate');

    var msPattern = /^ms-/;

    /**
     * 将小驼峰式书写的样式名转化为连字符式，特别将"ms-"起始的样式名转化成以"-ms-"起始
     * @param  {style} string "backgroundColor"、"ms-backgroundColor"等连字符拼接的样式名
     * @return {style}        连字符拼接的样式名
     */
    function hyphenateStyleName(string) {
      return hyphenate(string).replace(msPattern, '-ms-');
    }

    module.exports = hyphenateStyleName;