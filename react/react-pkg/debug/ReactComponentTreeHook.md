# ReactComponentTreeHook::react

## 概述

debug相关。

debug模式即本地开发时，用于获取节点的构建信息；通过ReactDebugTool注入组件实例化、挂载等过程中。

## 源码

    'use strict';
    
    // reactProdInvariant(code)，线上环境报错，提示facebook文档
    var _prodInvariant = require('./reactProdInvariant');
    
    // 缓存当前被实例化的内部组件实例，如ReactCompositeComponent等，用于调试、校验ReactElement等
    var ReactCurrentOwner = require('./ReactCurrentOwner');
    
    var invariant = require('fbjs/lib/invariant');
    var warning = require('fbjs/lib/warning');
    
    // 测试函数fn是否浏览器原生函数
    function isNative(fn) {
      var funcToString = Function.prototype.toString;
      var hasOwnProperty = Object.prototype.hasOwnProperty;
      var reIsNative = RegExp(
        '^' + 
        funcToString.call(hasOwnProperty)
          .replace(/[\\^$.*+?()[\]{}|]/g, '\\$&')// \\^$.*+?()[\]{}|字符添加"\\"，转化成正则需要
          .replace(/hasOwnProperty|(function).*?(?=\\\()| for .+?(?=\\\])/g, '$1.*?') 
        + '$'
      );// 测试是否原生函数正则"function .*?() { [native code] }"
      try {
        var source = funcToString.call(fn);
        return reIsNative.test(source);
      } catch (err) {
        return false;
      }
    }
    
    // 判断浏览器是否支持Array.from、Map、Map.prototype.keys、Set、Set.prototype.keys
    var canUseCollections =
      typeof Array.from === 'function' &&
      typeof Map === 'function' && isNative(Map) &&
      Map.prototype != null && typeof Map.prototype.keys === 'function' && isNative(Map.prototype.keys) &&
      typeof Set === 'function' && isNative(Set) &&
      Set.prototype != null && typeof Set.prototype.keys === 'function' && isNative(Set.prototype.keys);
    
    var setItem;
    var getItem;
    var removeItem;
    var getItemIDs;
    var addRoot;
    var removeRoot;
    var getRootIDs;
    
    // debug模式下缓存挂载的节点与根节点，id自增1方式创建
    if (canUseCollections) {
      var itemMap = new Map();
      var rootIDSet = new Set();
    
      setItem = function (id, item) {
        itemMap.set(id, item);
      };
      getItem = function (id) {
        return itemMap.get(id);
      };
      removeItem = function (id) {
        itemMap['delete'](id);
      };
      getItemIDs = function () {
        return Array.from(itemMap.keys());
      };
    
      addRoot = function (id) {
        rootIDSet.add(id);
      };
      removeRoot = function (id) {
        rootIDSet['delete'](id);
      };
      getRootIDs = function () {
        return Array.from(rootIDSet.keys());
      };
    } else {
      var itemByKey = {};
      var rootByKey = {};
    
      var getKeyFromID = function (id) {
        return '.' + id;
      };
      var getIDFromKey = function (key) {
        return parseInt(key.substr(1), 10);
      };
    
      setItem = function (id, item) {
        var key = getKeyFromID(id);
        itemByKey[key] = item;
      };
      getItem = function (id) {
        var key = getKeyFromID(id);
        return itemByKey[key];
      };
      removeItem = function (id) {
        var key = getKeyFromID(id);
        delete itemByKey[key];
      };
      getItemIDs = function () {
        return Object.keys(itemByKey).map(getIDFromKey);
      };
    
      addRoot = function (id) {
        var key = getKeyFromID(id);
        rootByKey[key] = true;
      };
      removeRoot = function (id) {
        var key = getKeyFromID(id);
        delete rootByKey[key];
      };
      getRootIDs = function () {
        return Object.keys(rootByKey).map(getIDFromKey);
      };
    }
    
    var unmountedIDs = [];
    
    // 移除当前模块缓存的id节点信息的同时，并移除其下挂载的所有子节点缓存
    function purgeDeep(id) {
      var item = getItem(id);
      if (item) {
        var childIDs = item.childIDs;
    
        removeItem(id);
        childIDs.forEach(purgeDeep);
      }
    }
    
    function describeComponentFrame(name, source, ownerName) {
      return '\n    in ' + (name || 'Unknown') + 
        (source ? 
          ' (at ' + source.fileName.replace(/^.*[\\\/]/, '') + ':' + source.lineNumber + ')' : 
          ownerName ? ' (created by ' + ownerName + ')' : '');
    }
    
    // 获取节点相应组件的标识符
    function getDisplayName(element) {
      if (element == null) {
        return '#empty';
      } else if (typeof element === 'string' || typeof element === 'number') {
        return '#text';
      } else if (typeof element.type === 'string') {
        return element.type;
      } else {
        return element.type.displayName || element.type.name || 'Unknown';
      }
    }
    
    // ReactElement元素的构建信息，其中_source属性由JSX注入
    function describeID(id) {
      var name = ReactComponentTreeHook.getDisplayName(id);
      var element = ReactComponentTreeHook.getElement(id);
      var ownerID = ReactComponentTreeHook.getOwnerID(id);
      var ownerName;
      if (ownerID) {
        ownerName = ReactComponentTreeHook.getDisplayName(ownerID);
      }
      process.env.NODE_ENV !== 'production' ? 
        warning(element, 'ReactComponentTreeHook: Missing React element for debugID %s when ' 
          + 'building stack', id) : void 0;
      return describeComponentFrame(name, element && element._source, ownerName);
    }
    
    // debug模式即本地开发时，用于获取节点的构建信息；通过ReactDebugTool注入组件实例化、挂载等过程中
    var ReactComponentTreeHook = {
      onSetChildren: function (id, nextChildIDs) {
        var item = getItem(id);
        !item ? process.env.NODE_ENV !== 'production' ? 
          invariant(false, 'Item must have been set') : _prodInvariant('144') : void 0;
        item.childIDs = nextChildIDs;
    
        for (var i = 0; i < nextChildIDs.length; i++) {
          var nextChildID = nextChildIDs[i];
          var nextChild = getItem(nextChildID);
          !nextChild ? process.env.NODE_ENV !== 'production' ? 
            invariant(false, 'Expected hook events to fire for the child before its parent includes it in onSetChildren().') 
            : _prodInvariant('140') : void 0;
          !(nextChild.childIDs != null || typeof nextChild.element !== 'object' || 
            nextChild.element == null) ? process.env.NODE_ENV !== 'production' ? 
            invariant(false, 'Expected onSetChildren() to fire for a container child before its parent includes it in onSetChildren().') 
            : _prodInvariant('141') : void 0;
          !nextChild.isMounted ? process.env.NODE_ENV !== 'production' ? 
            invariant(false, 'Expected onMountComponent() to fire for the child before its parent includes it in onSetChildren().') 
            : _prodInvariant('71') : void 0;
          if (nextChild.parentID == null) {
            nextChild.parentID = id;
          }
          !(nextChild.parentID === id) ? process.env.NODE_ENV !== 'production' ? 
            invariant(false, 'Expected onBeforeMountComponent() parent and onSetChildren() to be consistent (%s has parents %s and %s).', nextChildID, nextChild.parentID, id) 
            : _prodInvariant('142', nextChildID, nextChild.parentID, id) : void 0;
        }
      },
      onBeforeMountComponent: function (id, element, parentID) {
        var item = {
          element: element,
          parentID: parentID,
          text: null,
          childIDs: [],
          isMounted: false,
          updateCount: 0
        };
        setItem(id, item);
      },
      onBeforeUpdateComponent: function (id, element) {
        var item = getItem(id);
        if (!item || !item.isMounted) {
          return;
        }
        item.element = element;
      },
      onMountComponent: function (id) {
        var item = getItem(id);
        !item ? process.env.NODE_ENV !== 'production' ? 
          invariant(false, 'Item must have been set') : _prodInvariant('144') : void 0;
        item.isMounted = true;
        var isRoot = item.parentID === 0;
        if (isRoot) {
          addRoot(id);
        }
      },
      onUpdateComponent: function (id) {
        var item = getItem(id);
        if (!item || !item.isMounted) {
          return;
        }
        item.updateCount++;
      },
      onUnmountComponent: function (id) {
        var item = getItem(id);
        if (item) {
          item.isMounted = false;
          var isRoot = item.parentID === 0;
          if (isRoot) {
            removeRoot(id);
          }
        }
        unmountedIDs.push(id);
      },
      // 移除ReactComponentTreeHook所有已卸载节点及其子节点的缓存数据，缓存在当前模块中，即itemMap、itemByKey
      purgeUnmountedComponents: function () {
        if (ReactComponentTreeHook._preventPurging) {
          return;
        }
    
        for (var i = 0; i < unmountedIDs.length; i++) {
          var id = unmountedIDs[i];
          purgeDeep(id);
        }
        unmountedIDs.length = 0;
      },
      isMounted: function (id) {
        var item = getItem(id);
        return item ? item.isMounted : false;
      },
      getCurrentStackAddendum: function (topElement) {
        var info = '';
        if (topElement) {
          var name = getDisplayName(topElement);
          var owner = topElement._owner;
          info += describeComponentFrame(name, topElement._source, owner && owner.getName());
        }
    
        var currentOwner = ReactCurrentOwner.current;
        var id = currentOwner && currentOwner._debugID;
    
        info += ReactComponentTreeHook.getStackAddendumByID(id);
        return info;
      },
      getStackAddendumByID: function (id) {
        var info = '';
        while (id) {
          info += describeID(id);
          id = ReactComponentTreeHook.getParentID(id);
        }
        return info;
      },
      getChildIDs: function (id) {
        var item = getItem(id);
        return item ? item.childIDs : [];
      },
      getDisplayName: function (id) {
        var element = ReactComponentTreeHook.getElement(id);
        if (!element) {
          return null;
        }
        return getDisplayName(element);
      },
      getElement: function (id) {
        var item = getItem(id);
        return item ? item.element : null;
      },
      getOwnerID: function (id) {
        var element = ReactComponentTreeHook.getElement(id);
        if (!element || !element._owner) {
          return null;
        }
        return element._owner._debugID;
      },
      getParentID: function (id) {
        var item = getItem(id);
        return item ? item.parentID : null;
      },
      getSource: function (id) {
        var item = getItem(id);
        var element = item ? item.element : null;
        var source = element != null ? element._source : null;
        return source;
      },
      getText: function (id) {
        var element = ReactComponentTreeHook.getElement(id);
        if (typeof element === 'string') {
          return element;
        } else if (typeof element === 'number') {
          return '' + element;
        } else {
          return null;
        }
      },
      getUpdateCount: function (id) {
        var item = getItem(id);
        return item ? item.updateCount : 0;
      },
    
      getRootIDs: getRootIDs,
    
      // 获取已挂载的节点缓存数据
      getRegisteredIDs: getItemIDs
    };
    
    module.exports = ReactComponentTreeHook;