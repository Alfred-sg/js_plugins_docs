# accumulateInto::react-dom

## 概述

accumulate(current,next)，将current、next合并为数组。

## 源码

    'use strict';
    
    // reactProdInvariant(code)，线上环境报错，提示facebook文档
    var _prodInvariant = require('./reactProdInvariant');
    
    var invariant = require('fbjs/lib/invariant');
    
    // 合并，保留使用
    // 当首参为数组，次参为数组时，次参数组拼接到首参数组后
    // 当首参为数组，次参不是数组时，次参作为数组项添加到首参数组中
    // 当首参不是数组，次参是数组，首参转化为数组首项，并拼接次参数组
    function accumulateInto(current, next) {
      !(next != null) ? 
        process.env.NODE_ENV !== 'production' ? 
          invariant(false, 'accumulateInto(...): Accumulated items must not be null or undefined.') 
          : _prodInvariant('30') 
        : void 0;
    
      if (current == null) {
        return next;
      }
    
      if (Array.isArray(current)) {
        if (Array.isArray(next)) {
          current.push.apply(current, next);
          return current;
        }
        current.push(next);
        return current;
      }
    
      if (Array.isArray(next)) {
        return [current].concat(next);
      }
    
      return [current, next];
    }
    
    module.exports = accumulateInto;
    