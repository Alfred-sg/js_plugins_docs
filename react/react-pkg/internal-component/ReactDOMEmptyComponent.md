# ReactDOMEmptyComponent::react-dom

## 概述

挂载、更新或卸载空组件的管理容器组件。因实际绘制到页面中，需用ReactDOMComponentTree铰链ReactDOMEmptyComponent及其对应节点的引用关系。

## 源码

    'use strict';
    
    var _assign = require('object-assign');
    
    // react节点操作
    var DOMLazyTree = require('./DOMLazyTree');
    
    // 将ReactDOMComponent或ReactDOMTextComponent实例与其对应dom节点相互设置属性作引用
    var ReactDOMComponentTree = require('./ReactDOMComponentTree');
    
    var ReactDOMEmptyComponent = function (instantiate) {
      // ReactCompositeComponent uses this:
      this._currentElement = null;
      // ReactDOMComponentTree uses these:
      this._hostNode = null;
      this._hostParent = null;
      this._hostContainerInfo = null;
      this._domID = 0;
    };
    _assign(ReactDOMEmptyComponent.prototype, {
      mountComponent: function (transaction, hostParent, hostContainerInfo, context) {
        var domID = hostContainerInfo._idCounter++;
    
        // 实例的_domID用来校验节点的nodeValue文案是否匹配，以实现节点查询
        this._domID = domID;
        // 设置为父组件ReactDOMComponent实例，ReactDOMComponentTree.getNodeFromInstance方法中使用
        this._hostParent = hostParent;
        this._hostContainerInfo = hostContainerInfo;
    
        var nodeValue = ' react-empty: ' + this._domID + ' ';
        if (transaction.useCreateElement) {
          var ownerDocument = hostContainerInfo._ownerDocument;
          var node = ownerDocument.createComment(nodeValue);// 注释节点
    
          // 铰链ReactDOMEmptyComponent实例和node节点
          ReactDOMComponentTree.precacheNode(this, node);
          return DOMLazyTree(node);
        } else {
          if (transaction.renderToStaticMarkup) {
            return '';
          }
          return '<!--' + nodeValue + '-->';
        }
      },
      receiveComponent: function () {},
      getHostNode: function () {
        // 获取ReactDOMEmptyComponent实例对应的node节点
        return ReactDOMComponentTree.getNodeFromInstance(this);
      },
      unmountComponent: function () {
        // 解除ReactDOMEmptyComponent实例和node节点的铰链关系
        ReactDOMComponentTree.uncacheNode(this);
      }
    });
    
    module.exports = ReactDOMEmptyComponent;
    