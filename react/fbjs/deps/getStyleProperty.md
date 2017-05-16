# getStyleProperty

getStyleProperty(node,styleName)，获取node节点的styleName样式。

    'use strict';

    var camelize = require('./camelize');
    var hyphenate = require('./hyphenate');

    function asString(value){
      return value == null ? value : String(value);
    }


    // 获取node节点的name样式
    function getStyleProperty(node, name){
      var computedStyle = void 0;

      // W3C Standard
      if (window.getComputedStyle) {
        // In certain cases such as within an iframe in FF3, this returns null.
        computedStyle = window.getComputedStyle(node, null);
        if (computedStyle) {
          return asString(computedStyle.getPropertyValue(hyphenate(name)));
        }
      }
      // Safari
      if (document.defaultView && document.defaultView.getComputedStyle) {
        computedStyle = document.defaultView.getComputedStyle(node, null);
        // A Safari bug causes this to return null for `display: none` elements.
        if (computedStyle) {
          return asString(computedStyle.getPropertyValue(hyphenate(name)));
        }
        if (name === 'display') {
          return 'none';
        }
      }
      // Internet Explorer
      if (node.currentStyle) {
        if (name === 'float') {
          return asString(node.currentStyle.cssFloat || node.currentStyle.styleFloat);
        }
        return asString(node.currentStyle[camelize(name)]);
      }
      return asString(node.style && node.style[camelize(name)]);
    }

    module.exports = getStyleProperty;