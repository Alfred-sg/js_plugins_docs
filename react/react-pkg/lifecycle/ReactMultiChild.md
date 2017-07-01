# ReactMultiChild::react-dom

## 概述

子组件操控mixins。

挂载、更新或卸载ReactDomComponent组件实例下的子组件。调用ReactChildReconciler模块完成子组件的挂载、更新及卸载。

参数transaction管理生命周期方法的执行时机，对外提供transaction.getReactMountReady().enqueue(fn)添加生命周期方法。
参数context由父组件注入的context对象，构成当前组件实例的this.context属性。

* ReactMultiChild.Mixin.mountChildren(nestedChildren, transaction, context)，通过ReactReconciler.mountComponent获取props.children相关的ReactDomTree实例集合mountImages，数组形式。参数nestedChildren即props.children。
* ReactMultiChild.Mixin.updateTextContent(nextContent)，清空当前组件ReactDomComponent实例的内容后，设置该组件实例的文本。
* ReactMultiChild.Mixin.updateMarkup(nextMarkup)，清空当前组件ReactDomComponent实例的内容后，设置该组件实例的innerHTML。
* ReactMultiChild.Mixin.updateChildren(nextNestedChildrenElements, transaction, context)，视图层更新当前组件的子组件，或移动、或移除、或添加、或替换。
* ReactMultiChild.Mixin.unmountChildren(safely)，卸载子组件。

## 源码

    'use strict';
    
    // reactProdInvariant(code)，线上环境报错，提示facebook文档
    var _prodInvariant = require('./reactProdInvariant');
    
    // ReactComponentEnvironment.processChildrenUpdates(parentInst,[
    //  {type:"INSERT_MARKUP",content,afterNode,toIndex}, 
    //  {type:"MOVE_EXISTING",fromNode,afterNode,fromIndex,toIndex}, 
    //  {type:"SET_MARKUP",content}, {type:"TEXT_CONTENT",content}, 
    //  {type:"REMOVE_NODE",fromNode,fromIndex} 
    // ])
    //    根据type为ReactCompositeComponent实例关系节点插入子节点、移动子节点、设置节点innerHTML、设置节点文本、或移除子节点
    //    ReactDebugTool中的节点操作缓存记录为'insert child'、'move child'、'replace children'、'replace text'或'remove child'。
    // ReactComponentEnvironment.replaceNodeWithMarkup(oldChild,markup,prevInstance)
    //    将oldChild替换为markup；prevInstance影响ReactDebugTool中的节点操作缓存记录是'replace with'、还是'mount'
    var ReactComponentEnvironment = require('./ReactComponentEnvironment');
    
    // ReactInstanceMap.set(componentInst,internalInst)
    //    使用户构建的react组件实例componentInst的_reactInternalInstance属性指向react内部组件实例internalInst
    //    当外部组件挂载时，为该组件添加_reactInternalInstance属性
    // ReactInstanceMap.remove(componentInst)
    //    移除用户构建的react组件实例componentInst的_reactInternalInstance属性
    //    当外部组件卸载时，移除组件的_reactInternalInstance属性
    // ReactInstanceMap.get(componentInst)
    //    获取用户构建的react组件实例componentInst关联的react内部组件实例internalInst
    // ReactInstanceMap.has(componentInst)
    //  判断用户构建的react组件实例componentInst是否被挂载，含有_reactInternalInstance属性
    var ReactInstanceMap = require('./ReactInstanceMap');
    
    // ReactInstrumentation.debugTool，本地开发环境引用ReactDebugTool调试函数库
    var ReactInstrumentation = require('./ReactInstrumentation');
    
    // 缓存当前被实例化的内部组件实例，如ReactCompositeComponent等，用于调试、校验ReactElement
    var ReactCurrentOwner = require('react/lib/ReactCurrentOwner');
    
    // 用于挂载组件
    var ReactReconciler = require('./ReactReconciler');
    
    // 用于装载、更新、或卸载ReactDomComponent子组件，子组件可以有多个；ReactReconciler只能挂载单个
    var ReactChildReconciler = require('./ReactChildReconciler');
    
    var emptyFunction = require('fbjs/lib/emptyFunction');
    
    // flattenChildren(children,selfDebugID)，将子元素以对象形式{reactId: reactNode}输出
    var flattenChildren = require('./flattenChildren');
    
    var invariant = require('fbjs/lib/invariant');
    
    // 用于设置当前组件顶层节点的子节点
    function makeInsertMarkup(markup, afterNode, toIndex) {
      return {
        type: 'INSERT_MARKUP',
        content: markup,
        fromIndex: null,
        fromNode: null,
        toIndex: toIndex,
        afterNode: afterNode
      };
    }
        
    // 用于移动当前组件顶层节点的子节点
    function makeMove(child, afterNode, toIndex) {
      return {
        type: 'MOVE_EXISTING',
        content: null,
        fromIndex: child._mountIndex,
        fromNode: ReactReconciler.getHostNode(child),
        toIndex: toIndex,
        afterNode: afterNode
      };
    }
    
    // 用于移除当前组件顶层节点的子节点node
    function makeRemove(child, node) {
      return {
        type: 'REMOVE_NODE',
        content: null,
        fromIndex: child._mountIndex,
        fromNode: node,
        toIndex: null,
        afterNode: null
      };
    }
    
    // 用于设置当前组件顶层节点的innerHtml为markup
    function makeSetMarkup(markup) {
      return {
        type: 'SET_MARKUP',
        content: markup,
        fromIndex: null,
        fromNode: null,
        toIndex: null,
        afterNode: null
      };
    }
    
    // 用于设置当前组件顶层节点的文本为textContent
    function makeTextContent(textContent) {
      return {
        type: 'TEXT_CONTENT',
        content: textContent,
        fromIndex: null,
        fromNode: null,
        toIndex: null,
        afterNode: null
      };
    }
    
    // 队列形式缓存节点操作
    function enqueue(queue, update) {
      if (update) {
        queue = queue || [];
        queue.push(update);
      }
      return queue;
    }
    
    // 在文档中兑现队列updateQueue的节点操作
    // 通过ReactComponentEnvironment.processChildrenUpdates间接调用DOMChildrenOperations.processUpdates方法
    // 根据updateQueue元素项的type属性，为inst实例的关系节点执行不同的操作，或者设置子节点、节点文本、节点html或移除节点等
    function processQueue(inst, updateQueue) {
      ReactComponentEnvironment.processChildrenUpdates(inst, updateQueue);
    }
    
    // 将设置子节点的操作信息缓存到ReactDebugTool等关联缓存中
    var setChildrenForInstrumentation = emptyFunction;
    if (process.env.NODE_ENV !== 'production') {
      var getDebugID = function (inst) {
        if (!inst._debugID) {
          var internal;
          if (internal = ReactInstanceMap.get(inst)) {
            inst = internal;
          }
        }
        return inst._debugID;
      };
      setChildrenForInstrumentation = function (children) {
        var debugID = getDebugID(this);
        if (debugID !== 0) {
          ReactInstrumentation.debugTool.onSetChildren(debugID, children ? Object.keys(children).map(function (key) {
            return children[key]._debugID;
          }) : []);
        }
      };
    }
    
    var ReactMultiChild = {
    
      // 参数transaction管理生命周期方法的执行时机
      //    对外提供transaction.getReactMountReady().enqueue(fn)添加生命周期方法
      // 参数context由父组件注入的context对象，构成当前组件实例的this.context属性
      
      Mixin: {
        // 通过ReactChildReconciler.instantiateChildren获取props.children相关的react组件实例集合，对象形式
        _reconcilerInstantiateChildren: function (nestedChildren, transaction, context) {
          if (process.env.NODE_ENV !== 'production') {
            var selfDebugID = getDebugID(this);
            if (this._currentElement) {
              try {
                ReactCurrentOwner.current = this._currentElement._owner;
                return ReactChildReconciler.instantiateChildren(nestedChildren, transaction, context, selfDebugID);
              } finally {
                ReactCurrentOwner.current = null;
              }
            }
          }
          return ReactChildReconciler.instantiateChildren(nestedChildren, transaction, context);
        },
    
        // 获取nextChildren待更新或挂载的子节点，对象形式；引用类型更新mountImages、removedNodes
        //    mountImages只包含待更新或添加的子组件；nextChildren包含下次重绘的所有子组件
        // 参数prevChildren组件中先前挂载的子组件，以key键作为标识，用于更新或移除
        // 参数nextNestedChildrenElements当前待挂载的子组件，扁平化后，与prevChildren比对，生成待更新的子组件
        // 参数mountImages更新为组件最终渲染的子节点图谱，以引用类型形式更新
        // 参数removedNodes更新为组件待移除的子节点，以引用类型形式更新
        _reconcilerUpdateChildren: function (prevChildren, nextNestedChildrenElements, mountImages, removedNodes, transaction, context) {
          var nextChildren;
          var selfDebugID = 0;
          if (process.env.NODE_ENV !== 'production') {
            selfDebugID = getDebugID(this);
            if (this._currentElement) {
              try {
                ReactCurrentOwner.current = this._currentElement._owner;
                nextChildren = flattenChildren(nextNestedChildrenElements, selfDebugID);
              } finally {
                ReactCurrentOwner.current = null;
              }
              ReactChildReconciler.updateChildren(prevChildren, nextChildren, mountImages, removedNodes, transaction, this, this._hostContainerInfo, context, selfDebugID);
              return nextChildren;
            }
          }
          nextChildren = flattenChildren(nextNestedChildrenElements, selfDebugID);
          ReactChildReconciler.updateChildren(prevChildren, nextChildren, mountImages, removedNodes, transaction, this, this._hostContainerInfo, context, selfDebugID);
          return nextChildren;
        },
    
        // 通过ReactReconciler.mountComponent获取props.children相关的ReactDomTree实例集合mountImages，数组形式
        // 参数nestedChildren即props.children
        mountChildren: function (nestedChildren, transaction, context) {
          var children = this._reconcilerInstantiateChildren(nestedChildren, transaction, context);
          this._renderedChildren = children;// 缓存前次挂载的子组件
    
          // 通过ReactReconciler.mountComponent挂载子组件实例，获取ReactDomTree实例集合mountImages，数组形式
          var mountImages = [];
          var index = 0;
          for (var name in children) {
            if (children.hasOwnProperty(name)) {
              var child = children[name];
              var selfDebugID = 0;
              if (process.env.NODE_ENV !== 'production') {
                selfDebugID = getDebugID(this);
              }
              var mountImage = ReactReconciler.mountComponent(child, transaction, this, this._hostContainerInfo, context, selfDebugID);
              child._mountIndex = index++;
              mountImages.push(mountImage);
            }
          }
    
          // 通过ReactDebugTool缓存节点操作信息
          if (process.env.NODE_ENV !== 'production') {
            setChildrenForInstrumentation.call(this, children);
          }
    
          return mountImages;
        },
    
        // 清空当前组件ReactDomComponent实例的内容后，设置该组件实例的文本
        updateTextContent: function (nextContent) {
          var prevChildren = this._renderedChildren;
    
          // 卸载子组件，并校验子组件已完成卸载
          ReactChildReconciler.unmountChildren(prevChildren, false);
          for (var name in prevChildren) {
            if (prevChildren.hasOwnProperty(name)) {
              !false ? process.env.NODE_ENV !== 'production' ? 
                invariant(false, 'updateTextContent called on non-empty component.') : 
                _prodInvariant('118') : void 0;
            }
          }
    
          // 获取节点操作updates，processQueue函数执行该节点操作
          var updates = [makeTextContent(nextContent)];
          processQueue(this, updates);
        },
    
        // 清空当前组件ReactDomComponent实例的内容后，设置该组件实例的innerHTML
        updateMarkup: function (nextMarkup) {
          var prevChildren = this._renderedChildren;
    
          ReactChildReconciler.unmountChildren(prevChildren, false);
          for (var name in prevChildren) {
            if (prevChildren.hasOwnProperty(name)) {
              !false ? process.env.NODE_ENV !== 'production' ? 
              invariant(false, 'updateTextContent called on non-empty component.') : 
              _prodInvariant('118') : void 0;
            }
          }
    
          var updates = [makeSetMarkup(nextMarkup)];
          processQueue(this, updates);
        },
    
        // 调用this._updateChildren方法，视图层更新当前组件的子组件
        updateChildren: function (nextNestedChildrenElements, transaction, context) {
          this._updateChildren(nextNestedChildrenElements, transaction, context);
        },
    
        // 视图层更新当前组件的子组件，或移动、或移除、或添加、或替换
        _updateChildren: function (nextNestedChildrenElements, transaction, context) {
          var prevChildren = this._renderedChildren;
          var removedNodes = {};
          var mountImages = [];
          var nextChildren = this._reconcilerUpdateChildren(prevChildren, nextNestedChildrenElements, mountImages, removedNodes, transaction, context);
          if (!nextChildren && !prevChildren) {
            return;
          }
          var updates = null;
          var name;
          var nextIndex = 0;
          var lastIndex = 0;
          var nextMountIndex = 0;
          var lastPlacedNode = null;
          for (name in nextChildren) {
            if (!nextChildren.hasOwnProperty(name)) {
              continue;
            }
            var prevChild = prevChildren && prevChildren[name];
            var nextChild = nextChildren[name];
    
            // 移动子组件
            if (prevChild === nextChild) {
              updates = enqueue(updates, this.moveChild(prevChild, lastPlacedNode, nextIndex, lastIndex));
              lastIndex = Math.max(prevChild._mountIndex, lastIndex);
              prevChild._mountIndex = nextIndex;
    
            // 添加或替换子组件
            } else {
              if (prevChild) {
                lastIndex = Math.max(prevChild._mountIndex, lastIndex);
              }
              updates = enqueue(updates, this._mountChildAtIndex(nextChild, mountImages[nextMountIndex], lastPlacedNode, nextIndex, transaction, context));
              nextMountIndex++;
            }
            nextIndex++;
            lastPlacedNode = ReactReconciler.getHostNode(nextChild);
          }
    
          // 移除子组件
          for (name in removedNodes) {
            if (removedNodes.hasOwnProperty(name)) {
              updates = enqueue(updates, this._unmountChild(prevChildren[name], removedNodes[name]));
            }
          }
    
          if (updates) {
            processQueue(this, updates);
          }
    
          this._renderedChildren = nextChildren;
    
          if (process.env.NODE_ENV !== 'production') {
            setChildrenForInstrumentation.call(this, nextChildren);
          }
        },
    
        // 卸载子组件
        unmountChildren: function (safely) {
          var renderedChildren = this._renderedChildren;
          ReactChildReconciler.unmountChildren(renderedChildren, safely);
          this._renderedChildren = null;
        },
    
        // 获取移动子组件的操作数据update，调用processQueue函数执行实际的节点操作；当前模块使用
        moveChild: function (child, afterNode, toIndex, lastIndex) {
          if (child._mountIndex < lastIndex) {
            return makeMove(child, afterNode, toIndex);
          }
        },
    
        // 获取创建子节点的操作数据update，调用processQueue函数执行实际的节点操作；当前模块使用
        createChild: function (child, afterNode, mountImage) {
          return makeInsertMarkup(mountImage, afterNode, child._mountIndex);
        },
    
        // 获取删除节点的操作数据update，调用processQueue函数执行实际的节点操作；当前模块使用
        removeChild: function (child, node) {
          return makeRemove(child, node);
        },
    
        // 获取创建子节点的操作数据update，调用processQueue函数执行实际的节点操作；当前模块使用
        _mountChildAtIndex: function (child, mountImage, afterNode, index, transaction, context) {
          child._mountIndex = index;
          return this.createChild(child, afterNode, mountImage);
        },
    
        // 获取删除节点的操作数据update，调用processQueue函数执行实际的节点操作；当前模块使用
        _unmountChild: function (child, node) {
          var update = this.removeChild(child, node);
          child._mountIndex = null;
          return update;
        }
    
      }
    
    };
    
    module.exports = ReactMultiChild;
