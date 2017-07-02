# DOMLazyTree::react-dom

## 概述

react节点操作。

* DOMLazyTree(node)，创建DOMLazyTree对象，插入子节点或设置innerHTML或设置text后，调用insertTreeBefore静态方法插入文档。IE8-11或Edge，先缓存子节点、innerHTML、text，等待insertTreeBefore静态方法执行时将子节点、innerHTML、text插入tree.node中，再将tree.node插入文档；其他浏览器，直接将子节点、innerHTML、text插入tree.node，insertTreeBefore静态方法执行时再将tree.node插入文档。
* insertTreeBefore(parentNode,tree,referenceNode)，将tree.node插入文档内的parentNode节点下，并为tree填充IE8-11或Edge浏览器下缓存的tree.children或tree.html或tree.text（其他浏览器子节点、innerHTML、text事先插入tree.node节点中）。
* replaceChildWithTree(oldNode,newTree)，将节点从oldNode替换成newTree.node，并为tree填充IE8-11或Edge浏览器下缓存tree.children或tree.html或tree.text。
* queueChild(tree,children)，设置tree.node节点的children。IE8-11或Edge，先存储于tree.children中，再等待insertTreeBefore方法执行时插入文档；其他浏览器直接将子节点插入tree.node，insertTreeBefore方法执行时只将tree.node插入文档。
* queueHTML(tree,html)，设置tree.node节点的innerHTML。IE8-11或Edge先存储于tree.html中，再等待insertTreeBefore方法执行时插入文档；其他浏览器直接将innerHTML插入tree.node，insertTreeBefore方法执行时只将tree.node插入文档。
* queueText(tree,text)，设置tree.node节点的文本。IE8-11或Edge先存储于tree.text中，再等待insertTreeBefore方法执行时插入文档；其他浏览器直接将text插入tree.node，insertTreeBefore方法执行时只将tree.node插入文档。

## 源码

    'use strict';
    
    // 节点命名空间的集合
    var DOMNamespaces = require('./DOMNamespaces');
    
    // 为节点添加innerHTML
    var setInnerHTML = require('./setInnerHTML');
    
    // createMicrosoftUnsafeLocalFunction(func)，装饰func，使func执行时禁用ie浏览器的安全过滤机制
    var createMicrosoftUnsafeLocalFunction = require('./createMicrosoftUnsafeLocalFunction');
    
    // 文本节点添加nodeValue，其他节点添加textContent或经html转义后的innerHTML
    var setTextContent = require('./setTextContent');
    
    var ELEMENT_NODE_TYPE = 1;
    var DOCUMENT_FRAGMENT_NODE_TYPE = 11;
    
    // IE8-11或Edge下，向文档插入含子节点的节点要快于插入含子节点的节点
    var enableLazy = typeof document !== 'undefined' && typeof document.documentMode === 'number' || 
      typeof navigator !== 'undefined' && typeof navigator.userAgent === 'string' && 
      /\bEdge\/\d/.test(navigator.userAgent);
    
    // 将tree.children或tree.html或tree.text插入tree.node中
    // react创建节点的方式，由父节点注入子节点，或替换节点的innerHTML，或设置节点的文本
    function insertTreeChildren(tree) {
      if (!enableLazy) {
        return;
      }
      var node = tree.node;
      var children = tree.children;
      if (children.length) {
        for (var i = 0; i < children.length; i++) {
          insertTreeBefore(node, children[i], null);
        }
      } else if (tree.html != null) {
        setInnerHTML(node, tree.html);
      } else if (tree.text != null) {
        setTextContent(node, tree.text);
      }
    }
    
    // 将tree.node插入文档的parentNode节点下，并为tree填充tree.children或tree.html或tree.text
    var insertTreeBefore = createMicrosoftUnsafeLocalFunction(
      function (parentNode, tree, referenceNode) {
        // DocumentFragments、object标签如Flash Player，需要先插入子节点，然后再插入文档中
        if (tree.node.nodeType === DOCUMENT_FRAGMENT_NODE_TYPE 
          || tree.node.nodeType === ELEMENT_NODE_TYPE && tree.node.nodeName.toLowerCase() === 'object' 
          && (tree.node.namespaceURI == null || tree.node.namespaceURI === DOMNamespaces.html)) {
          insertTreeChildren(tree);
          parentNode.insertBefore(tree.node, referenceNode);
        } else {
          parentNode.insertBefore(tree.node, referenceNode);
          insertTreeChildren(tree);
        }
      }
    );
    
    // 将节点从oldNode替换成newTree.node，并为newTree填充newTree.children或newTree.html或newTree.text
    function replaceChildWithTree(oldNode, newTree) {
      oldNode.parentNode.replaceChild(newTree.node, oldNode);
      insertTreeChildren(newTree);
    }
    
    // IE8-11或Edge，将子节点存储在tree.children中，DOMLazyTree.insertTreeBefore方法调用时再行插入子节点
    // 其他浏览器，直接将子节点插入tree.node下
    function queueChild(parentTree, childTree) {
      if (enableLazy) {
        parentTree.children.push(childTree);
      } else {
        parentTree.node.appendChild(childTree.node);
      }
    }
    
    // IE8-11或Edge，将html存储在tree.html中，DOMLazyTree.insertTreeBefore方法调用时再行插入html
    // 其他浏览器，直接设置tree.node的innerHTML
    function queueHTML(tree, html) {
      if (enableLazy) {
        tree.html = html;
      } else {
        setInnerHTML(tree.node, html);
      }
    }
    
    // IE8-11或Edge，将text存储在tree.text中，DOMLazyTree.insertTreeBefore方法调用时再行插入text
    // 其他浏览器，直接将设置tree.node的文本
    function queueText(tree, text) {
      if (enableLazy) {
        tree.text = text;
      } else {
        setTextContent(tree.node, text);
      }
    }
    
    // 返回tree.node.nodeName
    function toString() {
      return this.node.nodeName;
    }
    
    function DOMLazyTree(node) {
      return {
        node: node,
        children: [],// IE8-11或Edge中，用以缓存tree.node的子节点，等待insertTreeBefore方法执行时插入文档
        html: null,// IE8-11或Edge中，用以设置tree.node的innerHTML，等待insertTreeBefore方法执行时插入文档
        text: null,// IE8-11或Edge中，用以设置tree.node的文本，等待insertTreeBefore方法执行时插入文档
        toString: toString// 返回tree.node的标签名
      };
    }
    
    // insertTreeBefore(parentNode,tree,referenceNode)将tree.node插入文档内的parentNode节点下
    // 并为tree填充tree.children或tree.html或tree.text
    DOMLazyTree.insertTreeBefore = insertTreeBefore;
    
    // replaceChildWithTree(oldNode,newTree)将节点从oldNode替换成newTree.node
    // 并为tree填充tree.children或tree.html或tree.text
    DOMLazyTree.replaceChildWithTree = replaceChildWithTree;
    
    // queueChild(tree,children)设置tree.node节点的children
    // IE8-11或Edge先存储于tree.children中，再等待insertTreeBefore方法执行时插入文档
    DOMLazyTree.queueChild = queueChild;
    
    // queueHTML(tree,html)设置tree.node节点的innerHTML
    // IE8-11或Edge先存储于tree.html中，再等待insertTreeBefore方法执行时插入文档
    DOMLazyTree.queueHTML = queueHTML;
    
    // queueText(tree,text)设置tree.node节点的文本
    // IE8-11或Edge先存储于tree.text中，再等待insertTreeBefore方法执行时插入文档
    DOMLazyTree.queueText = queueText;
    
    module.exports = DOMLazyTree;
