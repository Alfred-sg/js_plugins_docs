# accumulate::react-dom

## 概述

accumulate(current,next)，将current、next合并为数组。

## 源码

    'use strict';
    
    // reactProdInvariant(code)，线上环境报错，提示facebook文档
    var _prodInvariant = require('./reactProdInvariant');
    
    var invariant = require('fbjs/lib/invariant');
    
    // accumulate(current,next)，将current、next合并为数组
    function accumulate(current, next) {
      !(next != null) ? process.env.NODE_ENV !== 'production' ? 
        invariant(false, 'accumulate(...): Accumulated items must be not be null or undefined.') : 
        _prodInvariant('29') : void 0;
    
      if (current == null) {
        return next;
      }
    
      if (Array.isArray(current)) {
        return current.concat(next);
      }
    
      if (Array.isArray(next)) {
        return [current].concat(next);
      }
    
      return [current, next];
    }
    
    module.exports = accumulate;