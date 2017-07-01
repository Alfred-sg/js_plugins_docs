# ReactElementSymbol::react::react-dom

## 概述

作为标识符，构建react元素ReactElement时，注入为该元素的$$typeof属性。该标识符可用于判断是否ReactElement元素。

## 源码

    'use strict';
    
    // 作为标识符，构建react元素ReactElement时，注入为该元素的$$typeof属性
    var REACT_ELEMENT_TYPE = typeof Symbol === 'function' && Symbol['for'] && Symbol['for']('react.element') || 0xeac7;
    
    module.exports = REACT_ELEMENT_TYPE;