# getIteratorFn::react-dom

## 概述

工具函数

获取迭代器。

    var iteratorFn = getIteratorFn(myIterable);
    if (iteratorFn) {
      var iterator = iteratorFn.call(myIterable);
      ...
    }

## 源码

    'use strict';
    
    var ITERATOR_SYMBOL = typeof Symbol === 'function' && Symbol.iterator;
    var FAUX_ITERATOR_SYMBOL = '@@iterator'; 
    
    // 获取迭代器
    function getIteratorFn(maybeIterable) {
      var iteratorFn = maybeIterable && (ITERATOR_SYMBOL && maybeIterable[ITERATOR_SYMBOL] || maybeIterable[FAUX_ITERATOR_SYMBOL]);
      if (typeof iteratorFn === 'function') {
        return iteratorFn;
      }
    }
    
    module.exports = getIteratorFn;