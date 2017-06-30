# traverseAllChildren::react-dom

## 概述

遍历子元素，操纵子元素，如实例化子元素、装饰子元素或将子元素以对象形式缓存到调试记录中。

* traverseAllChildren(children,(traverseContext,child,name)=>{},traverseContext)，遍历children，尾参traverseContext作为次参callback回调的首参，每次遍历后执行callback回调，name即reactId。reactId由父子元素的key值以特定字符分隔构成，无key值使用元素所在序号。

调用情况：
* react::ReactChildren模块中，traverseContext由ReactChildren.forEach(children,func,ctx)参数构成{func,ctx,count}，用于遍历子元素，并以上下文count执行func，func的参数为子元素和子元素的序号count。
* react-dom::flattenChildren中，traverseContext以对象形式存储reactId和子元素，并校验reactId是否相重，随后将对象形式的reactId和子元素通过ReactDebugTool记录到ReactComponentTreeHook中，供调试。
* react-dom::ReactChildReconciler中，traverseContext以对象形式存储reactId和元素实例，并校验元素的key值是否相重。

## 源码

    'use strict';
    
    // reactProdInvariant(code)，线上环境报错，提示facebook文档
    var _prodInvariant = require('./reactProdInvariant');
    
    // 缓存当前被实例化的内部组件实例，如ReactCompositeComponent等，用于调试、校验ReactElement
    var ReactCurrentOwner = require('./ReactCurrentOwner');
    
    // ReactElement元素$$typeof属性，作为标识符，可用于判断是否ReactElement元素
    var REACT_ELEMENT_TYPE = require('./ReactElementSymbol');
    
    // 获取迭代器
    var getIteratorFn = require('./getIteratorFn');
    
    // escape方法用于转义用户为组件定义的key属性，构成reactId；unescape反转义
    var KeyEscapeUtils = require('./KeyEscapeUtils');
    
    var invariant = require('fbjs/lib/invariant');
    var warning = require('fbjs/lib/warning');
    
    var SEPARATOR = '.';
    var SUBSEPARATOR = ':';
    
    var didWarnAboutMaps = false;
    
    // 当自定义组件设定key属性时，通过KeyEscapeUtils.escape转化key属性后输出；无则输出子元素的序号
    function getComponentKey(component, index) {
      if (component && typeof component === 'object' && component.key != null) {
        return KeyEscapeUtils.escape(component.key);
      }
    
      return index.toString(36);
    }
    
    // 当参数children为单节点形式时，以传参traverseContext、children、nameSoFar执行callback回调
    // 当参数children为多节点形式时，递归调用traverseAllChildrenImpl遍历子孙节点
    //    以传参traverseContext、child、nameSoFar执行callback回调
    // nameSoFar构成ReactNode的reactId属性
    function traverseAllChildrenImpl(children, nameSoFar, callback, traverseContext) {
      var type = typeof children;
    
      if (type === 'undefined' || type === 'boolean') {
        children = null;
      }
    
      // children为单元素形式，执行回调callback，callback传参为traverseContext、children、nameSoFar
      // 当调用traverseAllChildren函数注入的props.children为单一元素时，nameSoFar=SEPARATOR+getComponentKey(children,0)
      //    为多个元素，nameSoFar也构成父子元素深度嵌套形式，以SEPARATOR起始，SUBSEPARATOR作为父子元素的分隔符
      if (children === null || type === 'string' || type === 'number' ||
      type === 'object' && children.$$typeof === REACT_ELEMENT_TYPE) {
        callback(traverseContext, children,
        nameSoFar === '' ? SEPARATOR + getComponentKey(children, 0) : nameSoFar);
        return 1;
      }
    
      var child;
      var nextName;
      var subtreeCount = 0; // 统计子元素的数目
      var nextNamePrefix = nameSoFar === '' ? SEPARATOR : nameSoFar + SUBSEPARATOR;
    
      // children成数组形式，遍历子元素嵌套调用traverseAllChildrenImpl，以执行callback回调
      //    callback传参为traverseContext、child、nextName
      if (Array.isArray(children)) {
        for (var i = 0; i < children.length; i++) {
          child = children[i];
          nextName = nextNamePrefix + getComponentKey(child, i);
          subtreeCount += traverseAllChildrenImpl(child, nextName, callback, traverseContext);
        }
    
      // children为迭代器，遍历子元素嵌套调用traverseAllChildrenImpl，以执行callback回调
      //    callback传参为traverseContext、child、nextName
      } else {
        var iteratorFn = getIteratorFn(children);
        if (iteratorFn) {
          var iterator = iteratorFn.call(children);
          var step;
          if (iteratorFn !== children.entries) {
            var ii = 0;
            while (!(step = iterator.next()).done) {
              child = step.value;
              nextName = nextNamePrefix + getComponentKey(child, ii++);
              subtreeCount += traverseAllChildrenImpl(child, nextName, callback, traverseContext);
            }
          } else {
            // children为Map对象，iteratorFn迭代获取键和值(即ReactElement元素)，提示react实验性地支持Map形式的children
            if (process.env.NODE_ENV !== 'production') {
              var mapsAsChildrenAddendum = '';
              if (ReactCurrentOwner.current) {
                var mapsAsChildrenOwnerName = ReactCurrentOwner.current.getName();
                if (mapsAsChildrenOwnerName) {
                  mapsAsChildrenAddendum = ' Check the render method of `' + mapsAsChildrenOwnerName + '`.';
                }
              }
              process.env.NODE_ENV !== 'production' ? 
                warning(didWarnAboutMaps, 'Using Maps as children is not yet fully supported. It is an ' 
                  + 'experimental feature that might be removed. Convert it to a ' 
                  + 'sequence / iterable of keyed ReactElements instead.%s', mapsAsChildrenAddendum) 
                : void 0;
              didWarnAboutMaps = true;
            }
            while (!(step = iterator.next()).done) {
              var entry = step.value;
              if (entry) {
                child = entry[1];// entry[0]键，entry[1]值，即ReactElement元素
                nextName = nextNamePrefix + KeyEscapeUtils.escape(entry[0]) + SUBSEPARATOR + getComponentKey(child, 0);
                subtreeCount += traverseAllChildrenImpl(child, nextName, callback, traverseContext);
              }
            }
          }
    
        // children为不支持的对象形式，报错
        } else if (type === 'object') {
          var addendum = '';
          if (process.env.NODE_ENV !== 'production') {
            addendum = ' If you meant to render a collection of children, use an array ' 
              + 'instead or wrap the object using createFragment(object) from the ' 
              + 'React add-ons.';
            if (children._isReactElement) {
              addendum = ' It looks like you\'re using an element created by a different ' 
                + 'version of React. Make sure to use only one copy of React.';
            }
            if (ReactCurrentOwner.current) {
              var name = ReactCurrentOwner.current.getName();
              if (name) {
                addendum += ' Check the render method of `' + name + '`.';
              }
            }
          }
          var childrenString = String(children);
          !false ? process.env.NODE_ENV !== 'production' ? 
            invariant(false, 'Objects are not valid as a React child (found: %s).%s', 
              childrenString === '[object Object]' ? 
              'object with keys {' + Object.keys(children).join(', ') + '}' 
              : childrenString, addendum) 
            : _prodInvariant('31', childrenString === '[object Object]' ? 
              'object with keys {' + Object.keys(children).join(', ') + '}' : childrenString, addendum) 
            : void 0;
        }
      }
    
      return subtreeCount;
    }
    
    // 遍历children，执行callback回调，callback首参为traverseContext
    function traverseAllChildren(children, callback, traverseContext) {
      if (children == null) {
        return 0;
      }
    
      return traverseAllChildrenImpl(children, '', callback, traverseContext);
    }
    
    // traverseAllChildren(children,function callback(traverseContext,child,name){}, traverseContext)
    //    尾参traverseContext作为次参callback回调的参数
    // react::ReactChildren模块中，traverseContext由ReactChildren.forEach(children,func,ctx)参数构成{func,ctx,count}
    //    用于遍历子元素，并以上下文count执行func，func的参数为子元素和子元素的序号count
    // react-dom::flattenChildren中，traverseContext以对象形式存储reactId和子元素，并校验reactId是否相重
    //    随后将对象形式的reactId和子元素通过ReactDebugTool记录到ReactComponentTreeHook中，供调试
    // react-dom::ReactChildReconciler中，traverseContext以对象形式存储reactId和元素实例，并校验元素的key值是否相重
    module.exports = traverseAllChildren;
