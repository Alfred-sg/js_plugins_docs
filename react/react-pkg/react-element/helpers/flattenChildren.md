# flattenChildren::react::react-dom

## 概述

flattenChildren(children,selfDebugID)，将子元素以对象形式{reactId: reactNode}输出。

## 源码

    'use strict';
    
    // escape方法用于转义用户为组件定义的key属性，构成reactId；unescape反转义
    var KeyEscapeUtils = require('./KeyEscapeUtils');
    
    // traverseAllChildren(children,(traverseContext,child,name)=>{},traverseContext)，遍历children
    //    尾参traverseContext作为次参callback回调的首参，每次遍历后执行callback回调，name即reactId
    var traverseAllChildren = require('./traverseAllChildren');
    
    var warning = require('fbjs/lib/warning');
    
    var ReactComponentTreeHook;
    
    if (typeof process !== 'undefined' && process.env && process.env.NODE_ENV === 'test') {
      // debug模式即本地开发时，用于获取节点的构建信息；通过ReactDebugTool注入组件实例化、挂载等过程中
      // ReactComponentTreeHook模块用于辅助实现ReactDebugTool模块
      ReactComponentTreeHook = require('react/lib/ReactComponentTreeHook');
    }
    
    function flattenSingleChildIntoContext(traverseContext, child, name, selfDebugID) {
      if (traverseContext && typeof traverseContext === 'object') {
        var result = traverseContext;
        var keyUnique = result[name] === undefined;
        if (process.env.NODE_ENV !== 'production') {
          if (!ReactComponentTreeHook) {
            ReactComponentTreeHook = require('react/lib/ReactComponentTreeHook');
          }
          if (!keyUnique) {
            process.env.NODE_ENV !== 'production' ? 
              warning(false, 'flattenChildren(...): Encountered two children with the same key, ' 
                + '`%s`. Child keys must be unique; when two children share a key, only ' 
                + 'the first child will be used.%s', KeyEscapeUtils.unescape(name), 
                ReactComponentTreeHook.getStackAddendumByID(selfDebugID)) 
              : void 0;
          }
        }
        if (keyUnique && child != null) {
          result[name] = child;
        }
      }
    }
    
    // 遍历子元素，以对象形式输出{reactId:reactNode}，开发环境校验子元素的key值是否相冲
    // 并将对象形式的节点数据
    function flattenChildren(children, selfDebugID) {
      if (children == null) {
        return children;
      }
      var result = {};
    
      if (process.env.NODE_ENV !== 'production') {
        traverseAllChildren(children, function (traverseContext, child, name) {
          return flattenSingleChildIntoContext(traverseContext, child, name, selfDebugID);
        }, result);
      } else {
        traverseAllChildren(children, flattenSingleChildIntoContext, result);
      }
      return result;
    }
    
    module.exports = flattenChildren;
    