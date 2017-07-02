# deprecated::react::react-dom

## 概述

deprecated(fnName,newModule,newPackage,ctx,fn)，已移除的函数fn以上下文ctx执行时予以提示须调用newPackage包下的[newModule][fnName]方法。

## 源码

    'use strict';
    
    var _assign = require('object-assign');
    
    var warning = require('fbjs/lib/warning');
    
    // 已移除的函数fn以上下文ctx执行时予以提示须调用newPackage包下的[newModule][fnName]方法
    function deprecated(fnName, newModule, newPackage, ctx, fn) {
      var warned = false;
      if (process.env.NODE_ENV !== 'production') {
        var newFn = function () {
          process.env.NODE_ENV !== 'production' ? 
            warning(warned,'React.%s is deprecated. Please use %s.%s from require' + '(\'%s\') ' + 'instead.', 
              fnName, newModule, fnName, newPackage) : void 0;
          warned = true;
          return fn.apply(ctx, arguments);
        };
        _assign(newFn, fn);
    
        return newFn;
      }
    
      return fn;
    }
    
    module.exports = deprecated;
    