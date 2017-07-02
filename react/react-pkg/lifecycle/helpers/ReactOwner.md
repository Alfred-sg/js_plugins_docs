# ReactOwner::react-dom

## 概述

当父组件的ref设定为字符串时，ReactOwner模块辅助添加或移除父组件的ref引用。

* ReactOwner.addComponentAsRefTo(component,ref,owner)，为父组件添加this.refs[ref]引用。参数component为子组件相关ReactInternalComponent实例，参数ref为字符串，参数owner为父组件相关的ReactInternalComponent容器管理组件实例，须校验是否为ReactCompositeComponent实例。
* ReactOwner.removeComponentAsRefFrom(component,ref,owner)，当父组件的ref引用和子组件相匹配时，移除父组件的this.refs[ref]引用。

## 源码

    'use strict';
    
    // reactProdInvariant(code)，线上环境报错，提示facebook文档
    var _prodInvariant = require('./reactProdInvariant');
    
    var invariant = require('fbjs/lib/invariant');
    
    // 通过有无attachRef、detachRef方法，校验是否ReactCompositeComponent实例
    // ReactCompositeComponent实例若含有attachRef、detachRef方法，可作为父组件；ReactDomComponent实例没有，只能作为子组件
    function isValidOwner(object) {
      return !!(object && typeof object.attachRef === 'function' && typeof object.detachRef === 'function');
    }
    
    var ReactOwner = {
    
      // 为父组件添加this.refs[ref]引用，参数component为子组件相关ReactInternalComponent实例，参数ref为字符串
      // 参数owner为父组件相关的ReactInternalComponent容器管理组件实例，须校验是否为ReactCompositeComponent实例
      addComponentAsRefTo: function (component, ref, owner) {
        !isValidOwner(owner) ? process.env.NODE_ENV !== 'production' ? 
          invariant(false, 'addComponentAsRefTo(...): Only a ReactOwner can have refs.' 
            + ' You might be adding a ref to a component that was not created inside a component\'s' 
            + ' `render` method, or you have multiple copies of React loaded' 
            + ' (details: https://fb.me/react-refs-must-have-owner).') 
          : _prodInvariant('119') : void 0;
    
        owner.attachRef(ref, component);
      },
    
      // 当父组件的ref引用和子组件相匹配时，移除父组件的this.refs[ref]引用
      removeComponentAsRefFrom: function (component, ref, owner) {
        !isValidOwner(owner) ? process.env.NODE_ENV !== 'production' ? 
          invariant(false, 'addComponentAsRefTo(...): Only a ReactOwner can have refs.' 
            + ' You might be adding a ref to a component that was not created inside a component\'s' 
            + ' `render` method, or you have multiple copies of React loaded' 
            + ' (details: https://fb.me/react-refs-must-have-owner).') 
          : _prodInvariant('119') : void 0;
    
        var ownerPublicInstance = owner.getPublicInstance();// 子组件相关ReactComponent实例
    
        if (ownerPublicInstance && ownerPublicInstance.refs[ref] === component.getPublicInstance()) {
          owner.detachRef(ref);
        }
      }
    
    };
    
    module.exports = ReactOwner;
