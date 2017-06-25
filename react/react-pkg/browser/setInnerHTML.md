# setInnerHTML::react-dom

## 概述

setInnerHTML(node,html)，设置节点的innerHTML，并处理兼容性问题。

## 源码

    'use strict';
    
    var ExecutionEnvironment = require('fbjs/lib/ExecutionEnvironment');
    
    // 节点命名空间的集合
    var DOMNamespaces = require('./DOMNamespaces');
    
    var WHITESPACE_TEST = /^[ \r\n\t\f]/;// 空格符匹配
    var NONVISIBLE_TEST = /<(!--|link|noscript|meta|script|style)[ \r\n\t\f\/>]/;
    
    // createMicrosoftUnsafeLocalFunction(func)，装饰func，使func执行时禁用ie浏览器的安全过滤机制
    var createMicrosoftUnsafeLocalFunction = require('./createMicrosoftUnsafeLocalFunction');
    
    var reusableSVGContainer;
    
    var setInnerHTML = createMicrosoftUnsafeLocalFunction(function (node, html) {
      // 为svg节点或普通dom节点赋予html内容
      if (node.namespaceURI === DOMNamespaces.svg && !('innerHTML' in node)) {
        reusableSVGContainer = reusableSVGContainer || document.createElement('div');
        reusableSVGContainer.innerHTML = '<svg>' + html + '</svg>';
        var svgNode = reusableSVGContainer.firstChild;
        while (svgNode.firstChild) {
          node.appendChild(svgNode.firstChild);
        }
      } else {
        node.innerHTML = html;
      }
    });
    
    if (ExecutionEnvironment.canUseDOM) {
      var testElement = document.createElement('div');
      testElement.innerHTML = ' ';
    
      // 针对IE8更新节点时空白字符将表现出怪异
      if (testElement.innerHTML === '') {
        setInnerHTML = function (node, html) {
          // IE8更新节点时空白字符将表现出怪异，重新添加节点能处理空白字符的怪异表现
          if (node.parentNode) {
            node.parentNode.replaceChild(node, node);
          }
    
          // 包含空格或以"<"开头、包含NONVISIBLE_TEST中字符
          if (WHITESPACE_TEST.test(html) || html[0] === '<' && NONVISIBLE_TEST.test(html)) {
            // 节点的innerHTML前插入字符String.fromCharCode(0xFEFF)，作为node的首个节点firstChild
            // 再通过node.removeChild或textNode.deleteData方法移除该文本子节点
            // String.fromCharCode(0xFEFF)是出于对UglifyJS清除U+FEFF的考虑
            node.innerHTML = String.fromCharCode(0xFEFF) + html;
    
            var textNode = node.firstChild;
            if (textNode.data.length === 1) {
              node.removeChild(textNode);
            } else {
              textNode.deleteData(0, 1);
            }
          } else {
            node.innerHTML = html;
          }
        };
      }
      testElement = null;
    }
    
    // 设置节点的innerHTML
    module.exports = setInnerHTML;