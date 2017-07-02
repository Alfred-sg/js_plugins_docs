# ReactDOMComponentTree::react-dom

## 概述

将ReactDOMComponent或ReactDOMTextComponent实例与其对应dom节点相互设置属性作引用，并提供由dom节点查找ReactDOMComponent或ReactDOMTextComponent实例、由ReactDOMComponent或ReactDOMTextComponent实例查找dom节点的接口。

* ReactDOMComponentTree.precacheNode(inst,node)，若inst为ReactCompositeComponent实例，将自定义组件下首个ReactDomComponent或ReactDOMTextComponent实例和node节点作铰链；若inst为ReactDomComponent或ReactDOMTextComponent实例，直接将该实例同node节点作铰链。
* ReactDOMComponentTree.precacheChildNodes(inst,node)，将ReactDomComponent或ReactDOMTextComponent实例下子节点同其对应组件实例作铰链。
* ReactDOMComponentTree.uncacheNode(inst)，解除ReactDomComponent或ReactDOMTextComponent实例同对应其dom节点的铰链关系。
* ReactDOMComponentTree.getClosestInstanceFromNode(node)，获取node节点最近的ReactDomComponent或ReactDOMTextComponent实例。特别，当两者未作铰链时，向上获取首各铰链的节点，再对子孙节点作铰链，最终输出node节点最近的ReactDomComponent或ReactDOMTextComponent实例。
* ReactDOMComponentTree.getInstanceFromNode(node)，获取与node对应的ReactDomComponent或ReactDOMTextComponent实例。
* ReactDOMComponentTree.getNodeFromInstance(node)获取ReactDOMComponent或ReactDOMTextComponent实例对应的node节点。特别，当两者未作铰链时，将inst所在用户自定义组件下所有ReactDOMComponent或ReactDOMTextComponent实例同其对应node节点作铰链，最终输出ReactDOMComponent或ReactDOMTextComponent实例对应的node节点。

## 源码

    'use strict';
    
    // reactProdInvariant(code)，线上环境报错，提示facebook文档
    var _prodInvariant = require('./reactProdInvariant');
    
    var DOMProperty = require('./DOMProperty');
    
    // ReactDOMComponentFlags.hasCachedChildNodes = 1<<0，即1
    var ReactDOMComponentFlags = require('./ReactDOMComponentFlags');
    
    var invariant = require('fbjs/lib/invariant');
    
    var ATTR_NAME = DOMProperty.ID_ATTRIBUTE_NAME;
    var Flags = ReactDOMComponentFlags;
    
    var internalInstanceKey = '__reactInternalInstance$' + Math.random().toString(36).slice(2);
    
    // 校验节点的'data-reactid'属性或文本内容同ReactDomComponent或ReactDOMTextComponent实例的_domID属性匹配
    function shouldPrecacheNode(node, nodeID) {
      return node.nodeType === 1 && node.getAttribute(ATTR_NAME) === String(nodeID) || 
        node.nodeType === 8 && node.nodeValue === ' react-text: ' + nodeID + ' ' || 
        node.nodeType === 8 && node.nodeValue === ' react-empty: ' + nodeID + ' ';
    }
    
    // 若component为ReactCompositeComponent实例，或其其下首个ReactDomComponent或ReactDOMTextComponent实例
    // 若component为ReactDomComponent或ReactDOMTextComponent实例，原样输出
    function getRenderedHostOrTextFromComponent(component) {
      var rendered;
      // ReactCompositeComponent实例含_renderedComponent指向子组件实例；ReactDomComponent实例没有
      while (rendered = component._renderedComponent) {
        component = rendered;
      }
      return component;
    }
    
    // 若inst为ReactCompositeComponent实例，将自定义组件下首个ReactDomComponent或ReactDOMTextComponent实例和node节点作铰链
    // 若inst为ReactDomComponent或ReactDOMTextComponent实例，直接将该实例同node节点作铰链
    function precacheNode(inst, node) {
      var hostInst = getRenderedHostOrTextFromComponent(inst);
      hostInst._hostNode = node;
      node[internalInstanceKey] = hostInst;
    }
    
    // 解除ReactDomComponent或ReactDOMTextComponent实例同对应其dom节点的铰链关系
    // 参数inst为ReactDomComponent或ReactDOMTextComponent实例
    function uncacheNode(inst) {
      var node = inst._hostNode;
      if (node) {
        delete node[internalInstanceKey];
        inst._hostNode = null;
      }
    }
    
    // 将ReactDomComponent或ReactDOMTextComponent实例下子节点同其对应组件实例作铰链
    // 参数inst为ReactDomComponent或ReactDOMTextComponent实例，其_flags起始值为0，执行precacheChildNodes方法后为1
    // 参数node为ReactDomComponent或ReactDOMTextComponent实例的相关节点
    function precacheChildNodes(inst, node) {
      if (inst._flags & Flags.hasCachedChildNodes) {
        return;
      }
      var children = inst._renderedChildren;
      var childNode = node.firstChild;
      outer: for (var name in children) {
        if (!children.hasOwnProperty(name)) {
          continue;
        }
        var childInst = children[name];
        // ReactDomComponent或ReactDOMTextComponent实例的_domID属性设置为自定义父组件下的嵌套子组件序号，1起始
        var childID = getRenderedHostOrTextFromComponent(childInst)._domID;
        if (childID === 0) {
          continue;
        }
    
        // 在子节点中找到'data-reactid'属性或文本内容同ReactDomComponent或ReactDOMTextComponent实例的_domID属性匹配的节点
        // 将该节点同ReactDomComponent或ReactDOMTextComponent实例作铰链
        for (; childNode !== null; childNode = childNode.nextSibling) {
          if (shouldPrecacheNode(childNode, childID)) {
            precacheNode(childInst, childNode);
            continue outer;
          }
        }
    
        // 没有'data-reactid'属性或文本内容同ReactDomComponent或ReactDOMTextComponent实例的_domID属性匹配的节点，报错
        !false ? process.env.NODE_ENV !== 'production' ? 
          invariant(false, 'Unable to find element with ID %s.', childID) : 
          _prodInvariant('32', childID) : void 0;
      }
      inst._flags |= Flags.hasCachedChildNodes;
    }
    
    // 获取node节点最近的ReactDomComponent或ReactDOMTextComponent实例
    // 特别，当两者未作铰链时，向上获取首各铰链的节点，再对子孙节点作铰链
    function getClosestInstanceFromNode(node) {
      if (node[internalInstanceKey]) {
        return node[internalInstanceKey];
      }
    
      // 用户自定义组件内的dom节点向上遍历至该组件内顶层dom节点
      var parents = [];
      while (!node[internalInstanceKey]) {
        parents.push(node);
        if (node.parentNode) {
          node = node.parentNode;
        } else {
          return null;
        }
      }
    
      var closest;
      var inst;
      for (; node && (inst = node[internalInstanceKey]); node = parents.pop()) {
        closest = inst;
        if (parents.length) {
          precacheChildNodes(inst, node);
        }
      }
    
      return closest;
    }
    
    // 获取与node对应的ReactDomComponent或ReactDOMTextComponent实例
    function getInstanceFromNode(node) {
      var inst = getClosestInstanceFromNode(node);
      if (inst != null && inst._hostNode === node) {
        return inst;
      } else {
        return null;
      }
    }
    
    // 获取ReactDOMComponent或ReactDOMTextComponent实例对应的node节点
    // 特别，当两者未作铰链时，将inst所在用户自定义组件下所有ReactDOMComponent或ReactDOMTextComponent实例同其对应node节点作铰链
    function getNodeFromInstance(inst) {
      // ReactDomComponent或ReactDOMTextComponent实例尚未实例化，报错
      // 实例化后inst._hostNode属性将设为null；铰链node节点后，设为node节点
      !(inst._hostNode !== undefined) ? process.env.NODE_ENV !== 'production' ? 
        invariant(false, 'getNodeFromInstance: Invalid argument.') : _prodInvariant('33') : void 0;
    
      if (inst._hostNode) {
        return inst._hostNode;
      }
    
      var parents = [];
      while (!inst._hostNode) {
        parents.push(inst);
        // 用户自定义组件下的所有ReactDomComponent实例，其_hostParent指向父ReactDomComponent
        // ReactDomComponent实例居于自定义组件顶层，则其_hostParent为null
        !inst._hostParent ? process.env.NODE_ENV !== 'production' ? 
          invariant(false, 'React DOM tree root should always have a node reference.') : 
          _prodInvariant('34') : void 0;
        inst = inst._hostParent;
      }
    
      // 将自定义组件下的所有ReactDomComponent实例与其对应node节点作铰链
      for (; parents.length; inst = parents.pop()) {
        precacheChildNodes(inst, inst._hostNode);
      }
        
      return inst._hostNode;// inst此时即传参
    }
    
    var ReactDOMComponentTree = {
      getClosestInstanceFromNode: getClosestInstanceFromNode,
      getInstanceFromNode: getInstanceFromNode,
      getNodeFromInstance: getNodeFromInstance,
      precacheChildNodes: precacheChildNodes,
      precacheNode: precacheNode,
      uncacheNode: uncacheNode
    };
    
    module.exports = ReactDOMComponentTree;
