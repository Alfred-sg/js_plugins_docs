# ReactInstanceMap::react-dom

## 概述

使用户构建的react组件的_reactInternalInstance属性指向react内部组件实例，即ReactCompositeComponent实例。

* ReactInstanceMap.set(componentInst,internalInst)，使用户构建的react组件实例componentInst的_reactInternalInstance属性指向react内部组件实例internalInst。当外部组件挂载时，为该组件添加_reactInternalInstance属性。
* ReactInstanceMap.remove(componentInst)，移除用户构建的react组件实例componentInst的_reactInternalInstance属性。当外部组件卸载时，移除组件的_reactInternalInstance属性。
* ReactInstanceMap.get(componentInst)，获取用户构建的react组件实例componentInst关联的react内部组件实例internalInst。
* ReactInstanceMap.has(componentInst)，判断用户构建的react组件实例componentInst是否被挂载，含有_reactInternalInstance属性。

## 源码

    'use strict';
    
    // 参数key为用户构建的react组件实例，即ReactComponent实例或ReactDomComponent实例
    // 参数value为react内部组件实例，即ReactCompositeComponent实例
    var ReactInstanceMap = {
    
      // 移除用户构建的react组件实例的_reactInternalInstance属性
      // 当外部组件卸载时，为该组件移除_reactInternalInstance属性
      remove: function (key) {
        key._reactInternalInstance = undefined;
      },
    
      get: function (key) {
        return key._reactInternalInstance;
      },
    
      has: function (key) {
        return key._reactInternalInstance !== undefined;
      },
    
      // 使用户构建的react组件实例的_reactInternalInstance属性指向react内部组件实例
      // 当外部组件挂载时，为该组件添加_reactInternalInstance属性
      set: function (key, value) {
        key._reactInternalInstance = value;
      }
    
    };
    
    module.exports = ReactInstanceMap;
