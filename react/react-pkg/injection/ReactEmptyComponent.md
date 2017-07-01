# ReactEmptyComponent::react-dom

## 概述

空组件加载器。在ReactDefaultInjection模块中统一加载，默认加载ReactDOMEmptyComponent模块。调用ReactEmptyComponent.create方法可生成空组件实例。

* ReactEmptyComponent.injection((instantiate)=>{ return new ReactDOMEmptyComponent(instantiate) })，加载空组件的构造函数ReactDOMEmptyComponent模块。注入的函数实际意义是作为ReactEmptyComponent.create方法内的执行逻辑。
* ReactEmptyComponent.create(instantiate)，创建空组件实例。

## 源码

    'use strict';
    
    // 由ReactDefaultInjection模块默认加载ReactDOMEmptyComponent模块，生成空组件实例
    var emptyComponentFactory;
        
    var ReactEmptyComponentInjection = {
      injectEmptyComponentFactory: function (factory) {
        emptyComponentFactory = factory;
      }
    };
    
    var ReactEmptyComponent = {
      create: function (instantiate) {
        return emptyComponentFactory(instantiate);
      }
    };
    
    ReactEmptyComponent.injection = ReactEmptyComponentInjection;
    
    module.exports = ReactEmptyComponent;
