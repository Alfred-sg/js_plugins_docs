# createMicrosoftUnsafeLocalFunction::react-dom

## 概述

createMicrosoftUnsafeLocalFunction(func)，ie浏览器下，装饰func函数，使func函数执行时禁用html安全筛选(通常是拼接节点操作)；非ie浏览器下，直接返回func函数。

## 源码

    'use strict';
    
    // ie浏览器下，装饰func函数，使func函数执行时禁用html安全筛选(通常是拼接节点操作)
    // 非ie浏览器下，直接返回func函数
    var createMicrosoftUnsafeLocalFunction = function (func) {
      // MSApp.execUnsafeLocalFunction(func)，ie下装饰funct，使func执行时禁用html安全筛选，通常是拼接节点操作
      if (typeof MSApp !== 'undefined' && MSApp.execUnsafeLocalFunction) {
        return function (arg0, arg1, arg2, arg3) {
          MSApp.execUnsafeLocalFunction(function () {
            return func(arg0, arg1, arg2, arg3);
          });
        };
      } else {
        return func;
      }
    };
    
    module.exports = createMicrosoftUnsafeLocalFunction;