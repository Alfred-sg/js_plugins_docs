# ReactReconciler::react-dom

## 概述

以策略模式调用ReactInternalComponent容器管理组件实例的mountComponent、receiveComponent、performUpdateIfNecessary、unmountComponent方法，并添加ReactDebugTool缓存{reactId:{reactElement}}数据，更新组件的ref引用。

* ReactReconciler.mountComponent(internalInstance,transaction,hostParent,hostContainerInfo,context,parentDebugID)，挂载组件，通过执行ReactInternalComponent.mountComponent方法获取待渲染到文档中的markup(DomLazyTre实例或html字符串)。在组件挂载的过程中，执行组件的componentWillMount、componentDidMount生命周期方法；组件挂载完成后，执行attachRefs函数，添加组件实例的ref引用，attachRefs函数执行机制同componentDidMount。
* ReactReconciler.unmountComponent(internalInstance,safely)，调用ReactInternalComponent.unmountComponent方法卸载组件，并移除组件实例的ref引用。
* ReactReconciler.receiveComponent(internalInstance,nextElement,transaction,context)，调用ReactInternalComponent.receiveComponent更新组件，并移除组件实例的ref引用。ReactReconciler.receiveComponent方法的执行时机是当组件的props属性变更时。
* ReactReconciler.performUpdateIfNecessary(internalInstance,transaction,updateBatchNumber)，调用ReactInternalComponent.performUpdateIfNecessary更新组件。ReactReconciler.performUpdateIfNecessary方法的执行时机为组件调用过setState、forceUpdate方法。
* ReactReconciler.getHostNode(internalInstance)，获取组件顶层节点，如ReactDomComponent组件即当前节点，ReactCompositeComponent组件为组件顶层节点。

## 源码

    'use strict';
    
    // 创建、销毁、比对组件的ref引用
    var ReactRef = require('./ReactRef');
    
    // 本地开发环境调用ReactDebugTool调试函数库
    var ReactInstrumentation = require('./ReactInstrumentation');
    
    var warning = require('fbjs/lib/warning');
    
    // 创建reactElement的refs属性
    function attachRefs() {
      ReactRef.attachRefs(this, this._currentElement);
    }
    
    var ReactReconciler = {
      // 挂载组件，通过执行ReactInternalComponent.mountComponent方法获取待渲染到文档中的markup(DomLazyTre实例或html字符串)
      //    在组件挂载的过程中，执行组件的componentWillMount、componentDidMount生命周期方法
      //    组件挂载完成后，执行attachRefs函数，添加组件实例的ref引用；attachRefs函数执行机制同componentDidMount
      // 参数internalInstance为ReactInternalComponent容器管理组件实例，含ReactCompositeComponent、ReactDomComponent等
      // 参数transaction为ReactReconcileTransaction实例，用于挂载componentDidMount后置钩子的回调函数
      //    并注入为ReactComponent用户自定义组件的参数updater，挂载setState等方法
      //    ReactReconcileTransaction实例将队列化调控挂载函数的执行时机
      mountComponent: function (internalInstance, transaction, hostParent, hostContainerInfo, context, parentDebugID ) {
        if (process.env.NODE_ENV !== 'production') {
          if (internalInstance._debugID !== 0) {
            ReactInstrumentation.debugTool.onBeforeMountComponent(internalInstance._debugID, internalInstance._currentElement, parentDebugID);
          }
        }
    
        var markup = internalInstance.mountComponent(transaction, hostParent, hostContainerInfo, context, parentDebugID);
        
        // 向ReactReconcileTransaction实例的后置钩子队列添加attachRefs回调函数，组件绘制完成后执行
        if (internalInstance._currentElement && internalInstance._currentElement.ref != null) {
          transaction.getReactMountReady().enqueue(attachRefs, internalInstance);
        }
    
        if (process.env.NODE_ENV !== 'production') {
          if (internalInstance._debugID !== 0) {
            ReactInstrumentation.debugTool.onMountComponent(internalInstance._debugID);
          }
        }
        return markup;
      },
    
      // 获取组件顶层节点，如ReactDomComponent组件即当前节点，ReactCompositeComponent组件为组件顶层节点
      getHostNode: function (internalInstance) {
        return internalInstance.getHostNode();
      },
    
      // 调用ReactInternalComponent.unmountComponent方法卸载组件，并移除组件实例的ref引用
      unmountComponent: function (internalInstance, safely) {
        if (process.env.NODE_ENV !== 'production') {
          if (internalInstance._debugID !== 0) {
            ReactInstrumentation.debugTool.onBeforeUnmountComponent(internalInstance._debugID);
          }
        }
    
        // 移除组件实例的ref引用
        ReactRef.detachRefs(internalInstance, internalInstance._currentElement);
    
        internalInstance.unmountComponent(safely);
    
        if (process.env.NODE_ENV !== 'production') {
          if (internalInstance._debugID !== 0) {
            ReactInstrumentation.debugTool.onUnmountComponent(internalInstance._debugID);
          }
        }
      },
    
      // 调用ReactInternalComponent.receiveComponent更新组件，并移除组件实例的ref引用
      // ReactReconciler.receiveComponent方法的执行时机是当组件的props属性变更时
      receiveComponent: function (internalInstance, nextElement, transaction, context) {
        var prevElement = internalInstance._currentElement;
    
        if (nextElement === prevElement && context === internalInstance._context) {
          return;
        }
    
        if (process.env.NODE_ENV !== 'production') {
          if (internalInstance._debugID !== 0) {
            ReactInstrumentation.debugTool.onBeforeUpdateComponent(internalInstance._debugID, nextElement);
          }
        }
    
        // 判断组件实例的ref引用是否需要更新
        var refsChanged = ReactRef.shouldUpdateRefs(prevElement, nextElement);
    
        // 移除组件实例的ref引用
        if (refsChanged) {
          ReactRef.detachRefs(internalInstance, prevElement);
        }
    
        internalInstance.receiveComponent(nextElement, transaction, context);
    
        // 更新组件实例的ref引用
        if (refsChanged && internalInstance._currentElement && internalInstance._currentElement.ref != null) {
          transaction.getReactMountReady().enqueue(attachRefs, internalInstance);
        }
    
        if (process.env.NODE_ENV !== 'production') {
          if (internalInstance._debugID !== 0) {
            ReactInstrumentation.debugTool.onUpdateComponent(internalInstance._debugID);
          }
        }
      },
    
      // 调用ReactInternalComponent.performUpdateIfNecessary更新组件
      // ReactReconciler.performUpdateIfNecessary方法的执行时机为组件调用过setState、forceUpdate方法
      performUpdateIfNecessary: function (internalInstance, transaction, updateBatchNumber) {
        // internalInstance._updateBatchNumber把组件添加到脏组件时+1，重绘
        // updateBatchNumber当ReactUpdates.flushBatchedUpdates方法执行时自增1
        // 意义是当组件被添加到脏组件的时候，及须重绘组件，这一过程通常由ReactUpdates.enqueueUpdate方法完成
        if (internalInstance._updateBatchNumber !== updateBatchNumber) {
          process.env.NODE_ENV !== 'production' ? 
            warning(internalInstance._updateBatchNumber == null || 
              internalInstance._updateBatchNumber === updateBatchNumber + 1, 
              'performUpdateIfNecessary: Unexpected batch number (current %s, ' + 'pending %s)', 
              updateBatchNumber, internalInstance._updateBatchNumber) 
            : void 0;
          return;
        }
    
        if (process.env.NODE_ENV !== 'production') {
          if (internalInstance._debugID !== 0) {
            ReactInstrumentation.debugTool.onBeforeUpdateComponent(internalInstance._debugID, internalInstance._currentElement);
          }
        }
    
        // internalInstance即ReactInternalComponent实例(含ReactDomComponent、ReactCompositeComponent组件实例)
        //    实例内含_pendingElement、_pendingStateQueue、_pendingForceUpdate属性用以判断更新方式
        // _pendingStateQueue为state数据变化引起，由this.setState方法发起
        // _pendingForceUpdate为调用this.forceUpdate方法发起
        internalInstance.performUpdateIfNecessary(transaction);
    
        if (process.env.NODE_ENV !== 'production') {
          if (internalInstance._debugID !== 0) {
            ReactInstrumentation.debugTool.onUpdateComponent(internalInstance._debugID);
          }
        }
      }
    
    };
    
    module.exports = ReactReconciler;
