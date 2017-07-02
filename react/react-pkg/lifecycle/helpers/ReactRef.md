# ReactRef::react-dom

## 概述

添加、变更或移除父组件的ref引用。在ReactReconciler模块挂载或更新组件时调用。

* ReactRef.attachRefs(instance,element)，为父组件element._owner添加ref引用。instance为子组件相关的ReactInternalComponent实例，element为子组件相关元素。
* ReactRef.detachRefs(instance,element)，移除父组件element._owner的ref引用。
* ReactRef.shouldUpdateRefs(prevElement,nextElement)，判断父组件的ref引用是否需要更新。

## 源码

    'use strict';
    
    // 调用顶层用户自定义组件的attachRef、detachRef方法，用于添加refs属性
    var ReactOwner = require('./ReactOwner');
    
    var ReactRef = {};
    
    // 为自定义组件ReactComponent实例添加ref引用，值为子组件实例
    // 参数ref为定义在reactElement元素的ref属性配置，字符串形式或函数形式
    //    字符串ref用于为owner添加this.refs[ref]属性
    //    函数(component)=>{this.C=component}用于为owner添加this.C属性
    // 参数component为子组件相关的ReactInternalComponent(ReactCompositeComponent、ReactDomComponent等)实例
    //    ReactInternalComponent.getPublicInstance获取自定义组件ReactComponent实例
    // 参数owner为子组件元素的reactElement._owner，即component的父组件相关ReactInternalComponent实例
    //    父组件render方法中绘制子组件时，调用React.createElement时通过ReactCurrentOwner.owner注入
    //    ReactCurrentOwner.owner记录正在挂载的ReactInternalComponent实例
    function attachRef(ref, component, owner) {
      if (typeof ref === 'function') {
        // component.getPublicInstance()获取ReactInternalComponent实例component相关的自定义组件ReactComponent实例
        ref(component.getPublicInstance());
      } else {
        // 调用父组件ReactInternalComponent实例owner的attachRef方法，为owner相关的自定义组件添加ref引用
        ReactOwner.addComponentAsRefTo(component, ref, owner);
      }
    }
    
    // 移除自定义组件的ref引用
    function detachRef(ref, component, owner) {
      if (typeof ref === 'function') {
        ref(null);
      } else {
        ReactOwner.removeComponentAsRefFrom(component, ref, owner);
      }
    }
    
    // 为父组件element._owner添加ref引用
    ReactRef.attachRefs = function (instance, element) {
      if (element === null || typeof element !== 'object') {
        return;
      }
      var ref = element.ref;// 用户设置的元素的ref属性，函数或者字符串
      if (ref != null) {
        // element._owner，
        attachRef(ref, instance, element._owner);
      }
    };
    
    // 当元素的ref属性为函数，其值变更时，更新父组件的ref引用
    // 或者父组件变更且ref为字符串时，更新父组件的ref引用；因ref引用在子组件挂载时添加，仅子组件变更无影响
    ReactRef.shouldUpdateRefs = function (prevElement, nextElement) {
      var prevRef = null;
      var prevOwner = null;
      if (prevElement !== null && typeof prevElement === 'object') {
        prevRef = prevElement.ref;
        prevOwner = prevElement._owner;
      }
    
      var nextRef = null;
      var nextOwner = null;
      if (nextElement !== null && typeof nextElement === 'object') {
        nextRef = nextElement.ref;
        nextOwner = nextElement._owner;
      }
    
      return prevRef !== nextRef || typeof nextRef === 'string' && nextOwner !== prevOwner;
    };
    
    // 移除父组件element._owner的ref引用
    ReactRef.detachRefs = function (instance, element) {
      if (element === null || typeof element !== 'object') {
        return;
      }
      var ref = element.ref;
      if (ref != null) {
        detachRef(ref, instance, element._owner);
      }
    };
    
    module.exports = ReactRef;
    