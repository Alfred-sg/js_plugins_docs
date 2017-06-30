# KeyEscapeUtils::react-dom

## 概述

escape方法用于转义用户为组件定义的key属性，构成reactId；unescape反转义。

## 源码

    'use strict';
    
    // 正则替换"="为"=0"、":"为"=2"，并未整个字符串加上"$"前缀；用以构成reactId
    function escape(key) {
      var escapeRegex = /[=:]/g;
      var escaperLookup = {
        '=': '=0',
        ':': '=2'
      };
      var escapedString = ('' + key).replace(escapeRegex, function (match) {
        return escaperLookup[match];
      });
    
      return '$' + escapedString;
    }
    
    // key以".$"起始，截取第二个字符后的字符；其他，截取第一个字符后的字符
    // 同时将截取后的字符内"=0"替换成"="、"=2"替换成":"
    function unescape(key) {
      var unescapeRegex = /(=0|=2)/g;
      var unescaperLookup = {
        '=0': '=',
        '=2': ':'
      };
      var keySubstring = key[0] === '.' && key[1] === '$' ? key.substring(2) : key.substring(1);
    
      return ('' + keySubstring).replace(unescapeRegex, function (match) {
        return unescaperLookup[match];
      });
    }
    
    var KeyEscapeUtils = {
      escape: escape,
      unescape: unescape
    };
    
    module.exports = KeyEscapeUtils;
    