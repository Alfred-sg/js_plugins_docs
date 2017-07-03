# ReactDOMIDOperations::react-dom

## 概述

react节点操作。

导出dangerouslyProcessChildrenUpdates方法，并提供给ReactComponentBrowserEnvironment模块。

* ReactDOMIDOperations.dangerouslyProcessChildrenUpdates(parentInst,[{type:"INSERT_MARKUP",content,afterNode,toIndex}, {type:"MOVE_EXISTING",fromNode,afterNode,fromIndex,toIndex}, {type:"SET_MARKUP",content}, {type:"TEXT_CONTENT",content}, {type:"REMOVE_NODE",fromNode,fromIndex} ])，根据type为ReactCompositeComponent实例关系节点插入子节点、移动子节点、设置节点innerHTML、设置节点文本、或移除子节点。ReactDebugTool中的节点操作缓存记录为'insert child'、'move child'、'replace children'、'replace text'或'remove child'。

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
    
    // ReactDOMComponentTree.getNodeFromInstance获取ReactDomComponent实例的对应节点
    var ReactDOMComponentTree = require('./ReactDOMComponentTree');
    
    var ReactDOMIDOperations = {
    
      // dangerouslyProcessChildrenUpdates(parentInst,[
      //  {type:"INSERT_MARKUP",content,afterNode,toIndex}, 
      //  {type:"MOVE_EXISTING",fromNode,afterNode,fromIndex,toIndex}, 
      //  {type:"SET_MARKUP",content}, 
      //  {type:"TEXT_CONTENT",content}, 
      //  {type:"REMOVE_NODE",fromNode,fromIndex} 
      // ])
      //    根据type为ReactCompositeComponent实例关系节点插入子节点、移动子节点、设置节点innerHTML、设置节点文本、或移除子节点
      //    ReactDebugTool中的节点操作缓存记录为'insert child'、'move child'、'replace children'、'replace text'或'remove child'
      dangerouslyProcessChildrenUpdates: function (parentInst, updates) {
        var node = ReactDOMComponentTree.getNodeFromInstance(parentInst);
        DOMChildrenOperations.processUpdates(node, updates);
      }
    };
    
    module.exports = ReactDOMIDOperations;
