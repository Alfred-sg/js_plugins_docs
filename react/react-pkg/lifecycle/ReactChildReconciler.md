# ReactChildReconciler::react-dom

## 概述

ReactMultiChild模块引用的工具库，用于管理多个子组件的装载、更新或卸载，由ReactMultiChild模块完成装载、更新或卸载动作。

* ReactChildReconciler.instantiateChildren(nestedChildNodes,transaction,context,selfDebugID)，实例化nestedChildNodes相关ReactInternalComponent组件，并以{reactId:ReactInternalComponent}对象形式输出。
* ReactChildReconciler.updateChildren(prevChildren,nextChildren,mountImages,removedNodes,transaction,hostParent,hostContainerInfo,context,selfDebugID)，比对前后两次已渲染或待渲染的reactNode，已渲染的reactNode存储在prevChild._currentElement中，实例化待渲染的reactNode，以{reactId:ReactInternalComponent}对象形式更新nextChildren，mountImages更新为待插入文档的节点，removedNodes更新为待移除的节点。在ReactMultiChild.Mixin._reconcilerUpdateChildren方法中调用，以及更新文档节点。
* ReactChildReconciler.unmountChildren卸载子组件。

## 源码

    'use strict';
    
    // ReactInternalComponent即ReactCompositeComponent、ReactDomComponent组件挂载、卸载、更新的策略调度器
    var ReactReconciler = require('./ReactReconciler');
    
    // 构建ReactInternalComponent管理容器组件实例，ReactInternalComponent包含ReactCompositeComponent、ReactDomComponent
    var instantiateReactComponent = require('./instantiateReactComponent');
    
    // escape方法用于转义用户为组件定义的key属性，构成reactId；unescape反转义
    var KeyEscapeUtils = require('./KeyEscapeUtils');
    
    // shouldUpdateReactComponent(prevElement,nextElement)，返回真值，重绘时更新组件实例；否值，销毁实例后重新创建
    // 组件的构造函数或key值不同，销毁实例后再行创建
    var shouldUpdateReactComponent = require('./shouldUpdateReactComponent');
    
    // traverseAllChildren(children,(traverseContext,child,name)=>{},traverseContext)，遍历children
    //    尾参traverseContext作为次参callback回调的首参，每次遍历后执行callback回调，name即reactId
    var traverseAllChildren = require('./traverseAllChildren');
    
    var warning = require('fbjs/lib/warning');
    
    var ReactComponentTreeHook;
    
    if (typeof process !== 'undefined' && process.env && process.env.NODE_ENV === 'test') {
      ReactComponentTreeHook = require('react/lib/ReactComponentTreeHook');
    }
    
    // 以集合childInstances获取ReactInternalComponent管理容器组件实例
    function instantiateChild(childInstances, child, name, selfDebugID) {
      var keyUnique = childInstances[name] === undefined;
      if (process.env.NODE_ENV !== 'production') {
        if (!ReactComponentTreeHook) {
          ReactComponentTreeHook = require('react/lib/ReactComponentTreeHook');
        }
        if (!keyUnique) {
          // ReactComponentTreeHook.getStackAddendumByID当key值相同，获取祖先节点的信息，警告用
          process.env.NODE_ENV !== 'production' ? 
            warning(false, 'flattenChildren(...): Encountered two children with the same key, ' 
              + '`%s`. Child keys must be unique; when two children share a key, only ' 
              + 'the first child will be used.%s', 
              KeyEscapeUtils.unescape(name), ReactComponentTreeHook.getStackAddendumByID(selfDebugID)) 
            : void 0;
        }
      }
      if (child != null && keyUnique) {
        childInstances[name] = instantiateReactComponent(child, true);
      }
    }
    
    // 用于装载、更新、或卸载ReactDomComponent子组件；被ReactMultiChild模块调用，用于管理ReactDomComponent的子元素
    var ReactChildReconciler = {
      // 创建nestedChildNodes相关ReactInternalComponent组件实例，并以对象形式输出
      instantiateChildren: function (nestedChildNodes, transaction, context, selfDebugID ) {
        if (nestedChildNodes == null) {
          return null;
        }
        var childInstances = {};
    
        if (process.env.NODE_ENV !== 'production') {
          traverseAllChildren(nestedChildNodes, function (childInsts, child, name) {
            return instantiateChild(childInsts, child, name, selfDebugID);
          }, childInstances);
        } else {
          traverseAllChildren(nestedChildNodes, instantiateChild, childInstances);
        }
        return childInstances;
      },
    
      // ReactMultiChild.Mixin._reconcilerUpdateChildren方法中调用，以引用对象形式更新nextChildren、mountImages、removedNodes
      //    nextChildren由ReactNode更新为ReactInternalComponent组件实例，供_reconcilerUpdateChildren更新文档节点
      // 参数prevChildren组件中前次挂载的子元素相关ReactInternalComponent容器管理组件实例，其_currentElement属性为挂载的元素
      //    对象形式，以reactId作为标识，同nextChildren比较后，用于更新或移除
      // 参数nextChildren组件中即将挂载的子元素，以reactId作为标识，同prevChildren比较后，用于添加或更新
      //    在updateChildren方法执行后更新为{reactId:ReactInternalComponent}对象
      // 参数mountImages为引用类型形式，在updateChildren方法执行后更新为组件最终渲染的子节点图谱
      // 参数removedNodes为引用类型形式，在updateChildren方法执行后更新为组件待移除的子节点
      // 参数transcation管理组件生命周期方法的执行时机
      // 参数context来自父组件的context，用于为子组件传递数据
      updateChildren: function (prevChildren, nextChildren, mountImages, removedNodes, transaction, hostParent, hostContainerInfo, context, selfDebugID ) {
        if (!nextChildren && !prevChildren) {
          return;
        }
        var name;
        var prevChild;
        for (name in nextChildren) {
          if (!nextChildren.hasOwnProperty(name)) {
            continue;
          }
          prevChild = prevChildren && prevChildren[name];
          var prevElement = prevChild && prevChild._currentElement;
          var nextElement = nextChildren[name];
    
          // 通过shouldUpdateReactComponent函数判断是否可以更新ReactDomComponent子组件；若不能，卸载后重新装载
          if (prevChild != null && shouldUpdateReactComponent(prevElement, nextElement)) {
            ReactReconciler.receiveComponent(prevChild, nextElement, transaction, context);
            nextChildren[name] = prevChild;
          } else {
            if (prevChild) {
              removedNodes[name] = ReactReconciler.getHostNode(prevChild);
              ReactReconciler.unmountComponent(prevChild, false);
            }
    
            var nextChildInstance = instantiateReactComponent(nextElement, true);
            nextChildren[name] = nextChildInstance;
    
            var nextChildMountImage = ReactReconciler.mountComponent(nextChildInstance, transaction, hostParent, hostContainerInfo, context, selfDebugID);
            mountImages.push(nextChildMountImage);
          }
        }
    
        for (name in prevChildren) {
          if (prevChildren.hasOwnProperty(name) && !(nextChildren && nextChildren.hasOwnProperty(name))) {
            prevChild = prevChildren[name];
            removedNodes[name] = ReactReconciler.getHostNode(prevChild);
            ReactReconciler.unmountComponent(prevChild, false);
          }
        }
      },
    
      // 卸载子组件
      unmountChildren: function (renderedChildren, safely) {
        for (var name in renderedChildren) {
          if (renderedChildren.hasOwnProperty(name)) {
            var renderedChild = renderedChildren[name];
            ReactReconciler.unmountComponent(renderedChild, safely);
          }
        }
      }
    };
    
    module.exports = ReactChildReconciler;
