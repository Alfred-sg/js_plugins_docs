# quoteAttributeValueForBrowser::react-dom

quoteAttributeValueForBrowser(value)，布尔型和数值型转化为字符串后输出；字符串经html转码，逐个字符处理["'&<>]；在最终输出值外层包裹双引号。

    'use strict';
    
    // 布尔型和数值型转化为字符串后输出；字符串经html转码，逐个字符处理["'&<>]
    var escapeTextContentForBrowser = require('./escapeTextContentForBrowser');
    
    // 转码后的字符串外加引号包裹
    function quoteAttributeValueForBrowser(value) {
      return '"' + escapeTextContentForBrowser(value) + '"';
    }
    
    module.exports = quoteAttributeValueForBrowser;