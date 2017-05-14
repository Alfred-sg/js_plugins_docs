# camelizeStyleName

camelizeStyleName(style)，将连字符书写的样式名转化为小驼峰式，特别将"-ms-"起始的样式名转化成以"ms"起始。

    'use strict';

    var camelize = require('./camelize');

    var msPattern = /^-ms-/;

    /**
     * 将连字符书写的样式名转化为小驼峰式，特别将"-ms-"起始的样式名转化成以"ms"起始
     * @param  {style} string "background-color"、"-ms-background-color"等连字符拼接的样式名
     * @return {style}        小驼峰式拼接的样式名
     */
    function camelizeStyleName(string) {
      return camelize(string.replace(msPattern, 'ms-'));
    }

    module.exports = camelizeStyleName;