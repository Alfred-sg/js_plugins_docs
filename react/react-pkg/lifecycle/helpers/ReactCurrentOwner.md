# ReactCurrentOwner::react

## 概述

debug相关。

缓存当前被实例化的内部组件实例，如ReactCompositeComponent等，用于调试、校验ReactElement等。特别，在ReactElement.createElement方法，父组件将以ReactCurrentOwner.current的形式传入子组件元素ReactElement._owner属性中，用于定位父组件对子组件的ref引用。

## 源码

    'use strict';
    
    // 缓存当前被实例化的内部组件实例，如ReactCompositeComponent等，用于调试、校验ReactElement
    var ReactCurrentOwner = {
      current: null
    };
    
    module.exports = ReactCurrentOwner;