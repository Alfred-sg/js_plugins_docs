# getElementRect

getElementRect(elem)，获取elem节点四周边际离页面上、左边际的距离，{top,bottom,left,right}形式。

    'use strict';

    var containsNode = require('./containsNode');

    // 获取节点上下左右边距离开页面上、左边际的距离，{top,bottom,left,right}形式
    function getElementRect(elem) {
      var docElem = document.documentElement;

      // FF 2, Safari 3 and Opera 9.5- do not support getBoundingClientRect().
      // IE9- will throw if the element is not in the document.
      if (!('getBoundingClientRect' in elem) || !containsNode(docElem, elem)) {
        return {
          left: 0,
          right: 0,
          top: 0,
          bottom: 0
        };
      }

      // getBoundingClientRect获取{top,bottom,left,right}为元素上下左右边距距离页面上、左边际的距离
      var rect = elem.getBoundingClientRect();

      // docElem.clientLeft、clientTop为元素四周边框的厚度，不指定边框或者元素不定位，为0
      return {
        left: Math.round(rect.left) - docElem.clientLeft,
        right: Math.round(rect.right) - docElem.clientLeft,
        top: Math.round(rect.top) - docElem.clientTop,
        bottom: Math.round(rect.bottom) - docElem.clientTop
      };
    }

    module.exports = getElementRect;