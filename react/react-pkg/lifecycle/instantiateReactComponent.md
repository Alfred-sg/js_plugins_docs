# instantiateReactComponent::react-dom

## 概述

instantiateReactComponent(node,shouldHaveDebugID)，构建ReactInternalComponent管理容器组件实例。ReactInternalComponent管理容器组件包含ReactCompositeComponent、ReactDomComponent、ReactDomTextComponent、ReactDomEmptyComponent等，用于挂载、更新或卸载ReactComponent组件。shouldHaveDebugID参数为真，ReactInternalComponent实例含有debugId属性。

## 源码

    'use strict';
    
    var _prodInvariant = require('./reactProdInvariant'),// reactProdInvariant(code)，线上环境报错，提示facebook文档
        _assign = require('object-assign');
    
    // new ReactCompositeComponent(element)，创建用户自定义组件实例
    var ReactCompositeComponent = require('./ReactCompositeComponent');
    
    // ReactEmptyComponent.create(instantiate)，默认通过调用ReactDOMEmptyComponent创建空组件实例
    var ReactEmptyComponent = require('./ReactEmptyComponent');
    
    // ReactHostComponent.createInstanceForText(text)，创建文本组件实例
    // ReactHostComponent.isTextComponent(component)，判断是否文本组件实例
    var ReactHostComponent = require('./ReactHostComponent');
    
    // 自增1生成debugId
    var getNextDebugID = require('./getNextDebugID');
    
    var invariant = require('fbjs/lib/invariant');
    var warning = require('fbjs/lib/warning');
    
    // 继承ReactCompositeComponent，并将instantiateReactComponent赋值给原型方法，避免循环依赖
    // 用于挂载用户自定义组件创建的元素
    var ReactCompositeComponentWrapper = function (element) {
      this.construct(element);
    };
    _assign(ReactCompositeComponentWrapper.prototype, ReactCompositeComponent, {
      _instantiateReactComponent: instantiateReactComponent
    });
    
    // 适用于提示require加载的文件没有export
    function getDeclarationErrorAddendum(owner) {
      if (owner) {
        var name = owner.getName();
        if (name) {
          return ' Check the render method of `' + name + '`.';
        }
      }
      return '';
    }
    
    // InternalComponent如ReactCompositeComponent、ReactDomComponent等，用于管理组件的挂载、更新和卸载
    // InternalComponent管理容器组件须包含mountComponent、receiveComponent
    function isInternalComponentType(type) {
      return typeof type === 'function' && typeof type.prototype !== 'undefined' && 
        typeof type.prototype.mountComponent === 'function' && 
        typeof type.prototype.receiveComponent === 'function';
    }
    
    // 构建ReactInternalComponent管理容器组件实例
    //    含ReactCompositeComponent、ReactDomComponent、ReactDomTextComponent、ReactDomEmptyComponent实例
    // 参数node为ReactNode，包含ReactElement(含ReactComponentElement、ReactDomElement)、ReactFragment、ReactText
    // 以ReactComponentElement为例，ReactComponentElement设置了构造函数ReactComponent、props
    //    ReactComponent中，render方法用于创建ReactComponentElement元素下含的ReactNode
    //    通过生成ReactCompositeComponent实例，调用该实例的mountComponent方法渲染到文档中
    function instantiateReactComponent(node, shouldHaveDebugID) {
      var instance;
    
      // 创建空组件实例
      if (node === null || node === false) {
        instance = ReactEmptyComponent.create(instantiateReactComponent);
    
      // React封装DOM标签组件、用户自定义组件处理
      } else if (typeof node === 'object') {
        var element = node;
        var type = element.type;
    
        // type类型不是函数或字符串时，提示require加载的文件没有export
        if (typeof type !== 'function' && typeof type !== 'string') {
          var info = '';
          if (process.env.NODE_ENV !== 'production') {
            if (type === undefined || typeof type === 'object' && 
              type !== null && Object.keys(type).length === 0) {
              info += ' You likely forgot to export your component from the file ' + 'it\'s defined in.';
            }
          }
          info += getDeclarationErrorAddendum(element._owner);
          !false ? process.env.NODE_ENV !== 'production' ? 
            invariant(false, 'Element type is invalid: expected a string (for built-in components) ' 
              + 'or a class/function (for composite components) but got: %s.%s', 
              type == null ? type : typeof type, info) 
            : _prodInvariant('130', type == null ? type : typeof type, info) : void 0;
        }
    
        // React封装DOM标签组件mountComponent、receiveComponent方法
        if (typeof element.type === 'string') {
          instance = ReactHostComponent.createInternalComponent(element);
        
        // element.type为含有mountComponent、receiveComponent方法的InternalComponent管理容器组件
        } else if (isInternalComponentType(element.type)) {
          instance = new element.type(element);
    
          // 维持旧有api的有效性
          if (!instance.getHostNode) {
            instance.getHostNode = instance.getNativeNode;
          }
    
        // 用户自定义组件，创建原型继承ReactCompositeComponent类的ReactCompositeComponentWrapper实例
        } else {
          instance = new ReactCompositeComponentWrapper(element);
        }
    
      // 文本组件
      } else if (typeof node === 'string' || typeof node === 'number') {
        instance = ReactHostComponent.createInstanceForText(node);
    
      } else {
        !false ? process.env.NODE_ENV !== 'production' ? 
          invariant(false, 'Encountered invalid React node of type %s', typeof node) : 
          _prodInvariant('131', typeof node) : void 0;
      }
    
      // 通过mountComponent、receiveComponent、getHostNode、unmountComponent方法校验InternalComponent管理容器组件
      if (process.env.NODE_ENV !== 'production') {
        process.env.NODE_ENV !== 'production' ? 
          warning(typeof instance.mountComponent === 'function' && 
            typeof instance.receiveComponent === 'function' && 
            typeof instance.getHostNode === 'function' && 
            typeof instance.unmountComponent === 'function', 'Only React Components can be mounted.') : void 0;
      }
    
      // 初始化参数
      instance._mountIndex = 0;
      instance._mountImage = null;
    
      // 携带debugId
      if (process.env.NODE_ENV !== 'production') {
        instance._debugID = shouldHaveDebugID ? getNextDebugID() : 0;
      }
    
      // Object.preventExtensions使对象属性不可扩展，但可修改
      if (process.env.NODE_ENV !== 'production') {
        if (Object.preventExtensions) {
          Object.preventExtensions(instance);
        }
      }
    
      return instance;
    }
    
    module.exports = instantiateReactComponent;
