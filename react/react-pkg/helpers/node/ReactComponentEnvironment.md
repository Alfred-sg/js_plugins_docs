# ReactComponentEnvironment::react-dom

## 概述

react节点操作。

通过ReactDefaultInjection模块加载ReactComponentBrowserEnvironment模块的processChildrenUpdates、replaceNodeWithMarkup方法。

* ReactComponentEnvironment.processChildrenUpdates(parentInst,[{type:"INSERT_MARKUP",content,afterNode,toIndex}, {type:"MOVE_EXISTING",fromNode,afterNode,fromIndex,toIndex}, {type:"SET_MARKUP",content}, {type:"TEXT_CONTENT",content}, {type:"REMOVE_NODE",fromNode,fromIndex} ])，根据type为ReactCompositeComponent实例关系节点插入子节点、移动子节点、设置节点innerHTML、设置节点文本、或移除子节点。ReactDebugTool中的节点操作缓存记录为'insert child'、'move child'、'replace children'、'replace text'或'remove child'。
* ReactComponentEnvironment.replaceNodeWithMarkup(oldChild,markup,prevInstance)，将oldChild替换为markup；prevInstance影响ReactDebugTool中的节点操作缓存记录是'replace with'、还是'mount'。
* ReactComponentEnvironment.injection.injectEnvironment(environment)，在ReactDefaultInjection模块中，默认加载ReactComponentBrowserEnvironment模块。

## 源码

    'use strict';
    
    // reactProdInvariant(code)，线上环境报错，提示facebook文档
    var _prodInvariant = require('./reactProdInvariant');
    
    var invariant = require('fbjs/lib/invariant');
    
    var injected = false;
    
    var ReactComponentEnvironment = {
    
      // replaceNodeWithMarkup(oldChild,markup,prevInstance)
      //    将oldChild替换为markup；prevInstance影响ReactDebugTool中的节点操作缓存记录是'replace with'、还是'mount'
      replaceNodeWithMarkup: null,
    
      // processChildrenUpdates.dangerouslyProcessChildrenUpdates(parentInst,[
      //  {type:"INSERT_MARKUP",content,afterNode,toIndex}, 
      //  {type:"MOVE_EXISTING",fromNode,afterNode,fromIndex,toIndex}, 
      //  {type:"SET_MARKUP",content}, 
      //  {type:"TEXT_CONTENT",content}, 
      //  {type:"REMOVE_NODE",fromNode,fromIndex} 
      // ])
      //    根据type为ReactCompositeComponent实例关系节点插入子节点、移动子节点、设置节点innerHTML、设置节点文本、或移除子节点
      //    ReactDebugTool中的节点操作缓存记录为'insert child'、'move child'、'replace children'、'replace text'或'remove child'
      processChildrenUpdates: null,
    
      injection: {
        // ReactDefaultInjection模块中，默认加载ReactComponentBrowserEnvironment模块
        injectEnvironment: function (environment) {
          !!injected ? process.env.NODE_ENV !== 'production' ? 
            invariant(false, 'ReactCompositeComponent: injectEnvironment() can only be called once.') : 
            _prodInvariant('104') : void 0;
          ReactComponentEnvironment.replaceNodeWithMarkup = environment.replaceNodeWithMarkup;
          ReactComponentEnvironment.processChildrenUpdates = environment.processChildrenUpdates;
          injected = true;
        }
      }
    
    };
    
    module.exports = ReactComponentEnvironment;
