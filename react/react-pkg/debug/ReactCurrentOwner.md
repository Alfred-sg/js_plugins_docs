# ReactCurrentOwner::react

## 概述

debug相关。

缓存当前被实例化的内部组件实例，如ReactCompositeComponent等，用于调试、校验ReactElement等。

## 源码

    'use strict';
    
    // 缓存当前被实例化的内部组件实例，如ReactCompositeComponent等，用于调试、校验ReactElement
    var ReactCurrentOwner = {
      current: null
    };
    
    module.exports = ReactCurrentOwner;