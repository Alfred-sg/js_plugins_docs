# ReactComponentBrowserEnvironment::react-dom

## 概述

react节点操作。

通过ReactDefaultInjection模块导出processChildrenUpdates、replaceNodeWithMarkup方法，并提供给ReactComponentEnvironment模块。

* ReactComponentBrowserEnvironment.processChildrenUpdates(parentInst,[{type:"INSERT_MARKUP",content,afterNode,toIndex}, {type:"MOVE_EXISTING",fromNode,afterNode,fromIndex,toIndex}, {type:"SET_MARKUP",content}, {type:"TEXT_CONTENT",content}, {type:"REMOVE_NODE",fromNode,fromIndex} ])，根据type为ReactCompositeComponent实例关系节点插入子节点、移动子节点、设置节点innerHTML、设置节点文本、或移除子节点。ReactDebugTool中的节点操作缓存记录为'insert child'、'move child'、'replace children'、'replace text'或'remove child'。
* ReactComponentBrowserEnvironment.replaceNodeWithMarkup(oldChild,markup,prevInstance)，将oldChild替换为markup；prevInstance影响ReactDebugTool中的节点操作缓存记录是'replace with'、还是'mount'。

## 源码

    'use strict';
    
    // dangerouslyReplaceNodeWithMarkup(oldChild,markup,prevInstance)
    //    将oldChild替换为markup；prevInstance影响ReactDebugTool中的节点操作缓存记录是'replace with'、还是'mount'
    // replaceDelimitedText(openingComment,closingComment,stringText)
    //    在注释节点openingComment、closingComment之间插入文本节点stringText，并移除注释节点间其他内容
    //    若stringText为否值，只移除注释节点openingComment、closingComment之间的内容
    //    ReactDebugTool中的节点操作缓存记录为'replace text'
    // processUpdates(parentNode,[
    //  {type:"INSERT_MARKUP",content,afterNode,toIndex}, 
    //  {type:"MOVE_EXISTING",fromNode,afterNode,fromIndex,toIndex}, 
    //  {type:"SET_MARKUP",content}, 
    //  {type:"TEXT_CONTENT",content}, 
    //  {type:"REMOVE_NODE",fromNode,fromIndex} 
    // ])
    //    根据type插入节点、移动节点、设置节点innerHTML、设置节点文本、或移除节点
    //    ReactDebugTool中的节点操作缓存记录为'insert child'、'move child'、'replace children'、'replace text'或'remove child'
    var DOMChildrenOperations = require('./DOMChildrenOperations');
    
    // ReactDOMIDOperations.dangerouslyProcessChildrenUpdates(parentInst,[
    //  {type:"INSERT_MARKUP",content,afterNode,toIndex}, 
    //  {type:"MOVE_EXISTING",fromNode,afterNode,fromIndex,toIndex}, 
    //  {type:"SET_MARKUP",content}, 
    //  {type:"TEXT_CONTENT",content}, 
    //  {type:"REMOVE_NODE",fromNode,fromIndex} 
    // ])
    //    根据type为ReactCompositeComponent实例关系节点插入子节点、移动子节点、设置节点innerHTML、设置节点文本、或移除子节点
    //    ReactDebugTool中的节点操作缓存记录为'insert child'、'move child'、'replace children'、'replace text'或'remove child'
    var ReactDOMIDOperations = require('./ReactDOMIDOperations');
    
    var ReactComponentBrowserEnvironment = {
    
      // processChildrenUpdates.dangerouslyProcessChildrenUpdates(parentInst,[
      //  {type:"INSERT_MARKUP",content,afterNode,toIndex}, 
      //  {type:"MOVE_EXISTING",fromNode,afterNode,fromIndex,toIndex}, 
      //  {type:"SET_MARKUP",content}, 
      //  {type:"TEXT_CONTENT",content}, 
      //  {type:"REMOVE_NODE",fromNode,fromIndex} 
      // ])
      //    根据type为ReactCompositeComponent实例关系节点插入子节点、移动子节点、设置节点innerHTML、设置节点文本、或移除子节点
      //    ReactDebugTool中的节点操作缓存记录为'insert child'、'move child'、'replace children'、'replace text'或'remove child'
      processChildrenUpdates: ReactDOMIDOperations.dangerouslyProcessChildrenUpdates,
    
      // replaceNodeWithMarkup(oldChild,markup,prevInstance)
      //    将oldChild替换为markup；prevInstance影响ReactDebugTool中的节点操作缓存记录是'replace with'、还是'mount'
      replaceNodeWithMarkup: DOMChildrenOperations.dangerouslyReplaceNodeWithMarkup
    
    };
    
    module.exports = ReactComponentBrowserEnvironment;
