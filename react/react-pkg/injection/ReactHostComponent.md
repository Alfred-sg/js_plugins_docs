# ReactHostComponent::react-dom

## 概述

Dom标签组件或其他平台的默认组件、文本组件的加载器。在ReactDefaultInjection模块中统一加载，默认加载ReactDOMComponent、ReactDOMTextComponent模块。

* ReactHostComponent.injection.injectGenericComponentClass(ReactDOMComponent)，加载Dom标签组件或其他平台的默认组件的构造函数。ReactDefaultInjection模块中默认加载ReactDOMComponent。
* ReactHostComponent.injection.injectTextComponentClass(ReactDOMTextComponent)，加载文本组件的构造函数。ReactDefaultInjection模块中默认加载ReactDOMTextComponent。
* ReactHostComponent.createInternalComponent(element)，创建Dom标签组件或其他平台的默认组件实例。
* ReactHostComponent.createInstanceForText(text)，创建文本组件实例。
* ReactHostComponent.isTextComponent(component)，判断是否文本组件实例。

## 源码

    'use strict';
    
    // reactProdInvariant(code)，线上环境报错，提示facebook文档
    var _prodInvariant = require('./reactProdInvariant');
    
    var invariant = require('fbjs/lib/invariant');
    
    var genericComponentClass = null;
    var textComponentClass = null;
    
    var ReactHostComponentInjection = {
      // 加载Dom标签组件或其他平台的默认组件的构造函数，加载器的设计用于适用不同平台，如浏览器、桌面应用
      injectGenericComponentClass: function (componentClass) {
        genericComponentClass = componentClass;
      },
      // 加载文本组件构造函数
      injectTextComponentClass: function (componentClass) {
        textComponentClass = componentClass;
      }
    };
    
    // 创建Dom标签组件或其他平台的默认组件
    function createInternalComponent(element) {
      !genericComponentClass ? process.env.NODE_ENV !== 'production' ? 
        invariant(false, 'There is no registered component for the tag %s', element.type) : 
        _prodInvariant('111', element.type) : void 0;
      return new genericComponentClass(element);
    }
    
    // 创建文本组件
    function createInstanceForText(text) {
      return new textComponentClass(text);
    }
    
    // 判断是否文本组件
    function isTextComponent(component) {
      return component instanceof textComponentClass;
    }
    
    var ReactHostComponent = {
      createInternalComponent: createInternalComponent,
      createInstanceForText: createInstanceForText,
      isTextComponent: isTextComponent,
      injection: ReactHostComponentInjection
    };
    
    module.exports = ReactHostComponent;
