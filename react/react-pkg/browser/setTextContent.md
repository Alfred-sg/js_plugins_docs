# setTextContent::react-dom

## 概述

setTextContent(node,text)，设置节点的文本内容，并处理兼容性问题。

## 源码

    'use strict';
    
    var ExecutionEnvironment = require('fbjs/lib/ExecutionEnvironment');
    
    // 布尔型和数值型转化为字符串后输出；字符串经html转码，逐个字符处理["'&<>]
    var escapeTextContentForBrowser = require('./escapeTextContentForBrowser');
    
    // 为节点添加innerHTML
    var setInnerHTML = require('./setInnerHTML');
    
    // 文本节点添加nodeValue，其他节点添加textContent
    var setTextContent = function (node, text) {
      if (text) {
        var firstChild = node.firstChild;
    
        if (firstChild && firstChild === node.lastChild && firstChild.nodeType === 3) {
          firstChild.nodeValue = text;
          return;
        }
      }
      node.textContent = text;
    };
    
    if (ExecutionEnvironment.canUseDOM) {
      // 文本节点添加nodeValue，其他节点添加转义后的innerHTML
      if (!('textContent' in document.documentElement)) {
        setTextContent = function (node, text) {
          if (node.nodeType === 3) {
            node.nodeValue = text;
            return;
          }
          setInnerHTML(node, escapeTextContentForBrowser(text));
        };
      }
    }
    
    module.exports = setTextContent;
