# ReactHostOperationHistoryHook::react-dom

## 概述

debug相关。

变更样式、节点、节点属性缓存操作信息；通过ReactDebugTool注入组件实例化、挂载等过程中。

## 源码

    'use strict';
    
    var history = [];
    
    // 变更样式、节点、节点属性缓存操作信息[{type,instanceID,payload}]，type为操作类型，instanceID为关联的ReactDomComponent实例，payload为操作内容；通过ReactDebugTool注入组件实例化、挂载等过程中
    var ReactHostOperationHistoryHook = {
      onHostOperation: function (operation) {
        history.push(operation);
      },
      // 清空当前模块缓存的样式、节点变更、节点属性变更操作信息
      clearHistory: function () {
        if (ReactHostOperationHistoryHook._preventClearing) {
          return;
        }
    
        history = [];
      },
      getHistory: function () {
        return history;
      }
    };
    
    module.exports = ReactHostOperationHistoryHook;