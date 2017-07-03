# DOMChildrenOperations::react-dom

## 概述

react节点操作。

* DOMChildrenOperations.dangerouslyReplaceNodeWithMarkup(oldChild,markup,prevInstance)，将oldChild替换为markup；prevInstance影响ReactDebugTool中的节点操作缓存记录是'replace with'、还是'mount'。
* DOMChildrenOperations.replaceDelimitedText(openingComment,closingComment,stringText)，在注释节点openingComment、closingComment之间插入文本节点stringText，并移除注释节点间其他内容；若stringText为否值，只移除注释节点openingComment、closingComment之间的内容。ReactDebugTool中的节点操作缓存记录为'replace text'。
* DOMChildrenOperations.processUpdates(parentNode,[{type:"INSERT_MARKUP",content,afterNode,toIndex}, {type:"MOVE_EXISTING",fromNode,afterNode,fromIndex,toIndex}, {type:"SET_MARKUP",content}, {type:"TEXT_CONTENT",content}, {type:"REMOVE_NODE",fromNode,fromIndex} ])根据type插入节点、移动节点、设置节点innerHTML、设置节点文本、或移除节点。ReactDebugTool中的节点操作缓存记录为'insert child'、'move child'、'replace children'、'replace text'或'remove child'。

## 源码

    'use strict';
    
    // react节点操作
    var DOMLazyTree = require('./DOMLazyTree');
    
    // Danger.dangerouslyReplaceNodeWithMarkup(oldChild,markup)，将oldChild替换成markup
    // markup可以是字符串或嵌套对象形式
    var Danger = require('./Danger');
    
    // ReactDOMComponentTree.getInstanceFromNode(node)
    //  获取与node对应的ReactDomComponent或ReactDOMTextComponent实例
    var ReactDOMComponentTree = require('./ReactDOMComponentTree');
    
    // 本地开发环境调用ReactDebugTool调试函数库
    var ReactInstrumentation = require('./ReactInstrumentation');
    
    // createMicrosoftUnsafeLocalFunction(func)，装饰func，使func执行时禁用ie浏览器的安全过滤机制
    var createMicrosoftUnsafeLocalFunction = require('./createMicrosoftUnsafeLocalFunction');
    
    // 设置节点的innetHTML
    var setInnerHTML = require('./setInnerHTML');
    
    // 文本节点添加nodeValue，其他节点添加textContent
    var setTextContent = require('./setTextContent');
    
    // 获取node[1]后的节点，或parentNode的子节点
    function getNodeAfter(parentNode, node) {
      if (Array.isArray(node)) {
        node = node[1];
      }
      return node ? node.nextSibling : parentNode.firstChild;
    }
    
    // 在parentNode节点下，在referenceNode节点前插入childNode
    // node.insertBefore方法的次参须为node下的子节点，首参可自定义
    var insertChildAt = createMicrosoftUnsafeLocalFunction(
      function (parentNode, childNode, referenceNode) {
        parentNode.insertBefore(childNode, referenceNode);
      }
    );
    
    // 在parentNode节点下，referenceNode节点前插入childTree.node节点，同时将childTree的子节点插入文档
    function insertLazyTreeChildAt(parentNode, childTree, referenceNode) {
      DOMLazyTree.insertTreeBefore(parentNode, childTree, referenceNode);
    }
    
    // 将childNode插入或移动到referenceNode节点前
    function moveChild(parentNode, childNode, referenceNode) {
      if (Array.isArray(childNode)) {
        moveDelimitedText(parentNode, childNode[0], childNode[1], referenceNode);
      } else {
        insertChildAt(parentNode, childNode, referenceNode);
      }
    }
    
    // childNode若为数组，移除parentNode节点下注释节点childNode[0]、childNode[0]间的节点，不包含childNode[0]
    // childNode若为节点，移除parentNode节点下的childNode节点
    function removeChild(parentNode, childNode) {
      if (Array.isArray(childNode)) {
        var closingComment = childNode[1];
        childNode = childNode[0];
        removeDelimitedText(parentNode, childNode, closingComment);
        parentNode.removeChild(closingComment);
      }
      parentNode.removeChild(childNode);
    }
    
    // 在referenceNode节点前插入openingComment、closingComment间所有节点，含openingComment、closingComment
    function moveDelimitedText(parentNode, openingComment, closingComment, referenceNode) {
      var node = openingComment;
      while (true) {
        var nextNode = node.nextSibling;
        insertChildAt(parentNode, node, referenceNode);
        if (node === closingComment) {
          break;
        }
        node = nextNode;
      }
    }
    
    // 移除parentNode节点下介于startNode节点和closingComment节点之间的节点
    function removeDelimitedText(parentNode, startNode, closingComment) {
      while (true) {
        var node = startNode.nextSibling;
        if (node === closingComment) {
          break;
        } else {
          parentNode.removeChild(node);
        }
      }
    }
    
    // 将stringText作为文本节点插入到注释节点openingComment、closingComment之间，并移除注释节点间其他内容
    // 若stringText为否值，只移除注释节点openingComment、closingComment间的内容
    // 通过ReactInstrumentation.debugTool调用ReactDebugTool记录节点变更操作
    function replaceDelimitedText(openingComment, closingComment, stringText) {
      var parentNode = openingComment.parentNode;
      var nodeAfterComment = openingComment.nextSibling;
      if (nodeAfterComment === closingComment) {
        if (stringText) {
          insertChildAt(parentNode, document.createTextNode(stringText), nodeAfterComment);
        }
      } else {
        if (stringText) {
          setTextContent(nodeAfterComment, stringText);
          removeDelimitedText(parentNode, nodeAfterComment, closingComment);
        } else {
          removeDelimitedText(parentNode, openingComment, closingComment);
        }
      }
    
      if (process.env.NODE_ENV !== 'production') {
        ReactInstrumentation.debugTool.onHostOperation({
          instanceID: ReactDOMComponentTree.getInstanceFromNode(openingComment)._debugID,
          type: 'replace text',
          payload: stringText
        });
      }
    }
    
    // 将oldChild替换为markup；并调用ReactDebugTool记录节点变更或挂载操作
    var dangerouslyReplaceNodeWithMarkup = Danger.dangerouslyReplaceNodeWithMarkup;
    if (process.env.NODE_ENV !== 'production') {
      dangerouslyReplaceNodeWithMarkup = function (oldChild, markup, prevInstance) {
        Danger.dangerouslyReplaceNodeWithMarkup(oldChild, markup);
        if (prevInstance._debugID !== 0) {
          ReactInstrumentation.debugTool.onHostOperation({
            instanceID: prevInstance._debugID,
            type: 'replace with',
            payload: markup.toString()
          });
        } else {
          var nextInstance = ReactDOMComponentTree.getInstanceFromNode(markup.node);
          if (nextInstance._debugID !== 0) {
            ReactInstrumentation.debugTool.onHostOperation({
              instanceID: nextInstance._debugID,
              type: 'mount',
              payload: markup.toString()
            });
          }
        }
      };
    }
    
    var DOMChildrenOperations = {
    
      // dangerouslyReplaceNodeWithMarkup(oldChild,markup,prevInstance)，将oldChild替换为markup
      dangerouslyReplaceNodeWithMarkup: dangerouslyReplaceNodeWithMarkup,
    
      // replaceDelimitedText(openingComment,closingComment,stringText)
      // 在注释节点openingComment、closingComment之间插入文本节点stringText，并移除注释节点间其他内容
      replaceDelimitedText: replaceDelimitedText,
    
      // (parentNode,[{type:"INSERT_MARKUP",content,afterNode}])根据type操作节点
      processUpdates: function (parentNode, updates) {
        if (process.env.NODE_ENV !== 'production') {
          var parentNodeDebugID = ReactDOMComponentTree.getInstanceFromNode(parentNode)._debugID;
        }
    
        for (var k = 0; k < updates.length; k++) {
          var update = updates[k];
          switch (update.type) {
            // 将update.content节点及其子节点插入到pdate.afterNode节点前或parent下首个子节点前
            case 'INSERT_MARKUP':
              insertLazyTreeChildAt(parentNode, update.content, getNodeAfter(parentNode, update.afterNode));
              if (process.env.NODE_ENV !== 'production') {
                ReactInstrumentation.debugTool.onHostOperation({
                  instanceID: parentNodeDebugID,
                  type: 'insert child',
                  payload: { toIndex: update.toIndex, content: update.content.toString() }
                });
              }
              break;
    
            // 将update.fromNode节点移动到update.afterNode后的兄弟节点或parent下首个子节点前
            case 'MOVE_EXISTING':
              moveChild(parentNode, update.fromNode, getNodeAfter(parentNode, update.afterNode));
              if (process.env.NODE_ENV !== 'production') {
                ReactInstrumentation.debugTool.onHostOperation({
                  instanceID: parentNodeDebugID,
                  type: 'move child',
                  payload: { fromIndex: update.fromIndex, toIndex: update.toIndex }
                });
              }
              break;
    
            // 设置节点parentNode的innerHTML值为update.content
            case 'SET_MARKUP':
              setInnerHTML(parentNode, update.content);
              if (process.env.NODE_ENV !== 'production') {
                ReactInstrumentation.debugTool.onHostOperation({
                  instanceID: parentNodeDebugID,
                  type: 'replace children',
                  payload: update.content.toString()
                });
              }
              break;
    
            // parentNode若为文本节点，添加nodeValue属性，若为其他节点添加textContent属性，值为update.content
            case 'TEXT_CONTENT':
              setTextContent(parentNode, update.content);
              if (process.env.NODE_ENV !== 'production') {
                ReactInstrumentation.debugTool.onHostOperation({
                  instanceID: parentNodeDebugID,
                  type: 'replace text',
                  payload: update.content.toString()
                });
              }
              break;
    
            // parentNode下移除update.fromNode节点
            case 'REMOVE_NODE':
              removeChild(parentNode, update.fromNode);
              if (process.env.NODE_ENV !== 'production') {
                ReactInstrumentation.debugTool.onHostOperation({
                  instanceID: parentNodeDebugID,
                  type: 'remove child',
                  payload: { fromIndex: update.fromIndex }
                });
              }
              break;
          }
        }
      }
    
    };
    
    module.exports = DOMChildrenOperations;
