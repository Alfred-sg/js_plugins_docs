# Danger::react-dom

## 概述

react节点操作。

Danger.dangerouslyReplaceNodeWithMarkup(oldChild,markup)，将oldChild替换成markup；markup可以是字符串或嵌套对象形式。

## 源码

    'use strict';
    
    // reactProdInvariant(code)，线上环境报错，提示facebook文档
    var _prodInvariant = require('./reactProdInvariant');
    
    // react节点操作
    var DOMLazyTree = require('./DOMLazyTree');
    
    var ExecutionEnvironment = require('fbjs/lib/ExecutionEnvironment');
    
    // createNodesFromMarkup(markup, handleScript)，markup以html形式设置子节点，并返回子节点node对象集合nodeList
    // handleScript为真值允许设置script节点，并以函数形式处理script节点
    var createNodesFromMarkup = require('fbjs/lib/createNodesFromMarkup');
    
    var emptyFunction = require('fbjs/lib/emptyFunction');
    var invariant = require('fbjs/lib/invariant');
    
    var Danger = {
    
      // 将oldChild替换成markup；markup可以是字符串或嵌套对象形式
      dangerouslyReplaceNodeWithMarkup: function (oldChild, markup) {
        !ExecutionEnvironment.canUseDOM ? 
          process.env.NODE_ENV !== 'production' ? 
            invariant(false, 'dangerouslyReplaceNodeWithMarkup(...): ' 
              + 'Cannot render markup in a worker thread. ' 
              + 'Make sure `window` and `document` are available globally before requiring React' 
              + ' when unit testing or use ReactDOMServer.renderToString() for server rendering.') 
            : _prodInvariant('56') 
          : void 0;
    
        !markup ? 
          process.env.NODE_ENV !== 'production' ? 
            invariant(false, 'dangerouslyReplaceNodeWithMarkup(...): Missing markup.') 
            : _prodInvariant('57') 
          : void 0;
    
        // 不能替换html节点
        !(oldChild.nodeName !== 'HTML') ? 
          process.env.NODE_ENV !== 'production' ? 
            invariant(false, 'dangerouslyReplaceNodeWithMarkup(...): ' 
              + 'Cannot replace markup of the <html> node. ' 
              + 'This is because browser quirks make this unreliable and/or slow. ' 
              + 'If you want to render to the root you must use server rendering. ' 
              + 'See ReactDOMServer.renderToString().') 
            : _prodInvariant('58') 
          : void 0;
    
        if (typeof markup === 'string') {
          var newChild = createNodesFromMarkup(markup, emptyFunction)[0];
          oldChild.parentNode.replaceChild(newChild, oldChild);
        } else {
          DOMLazyTree.replaceChildWithTree(oldChild, markup);
        }
      }
    
    };
    
    module.exports = Danger;