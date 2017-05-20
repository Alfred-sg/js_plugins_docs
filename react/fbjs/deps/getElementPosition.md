# getElementPosition

getElementPosition(elem)，获取elem节点距离页面上、左边际的距离，以及节点的宽、高。

    'use strict';

    var getElementRect = require('./getElementRect');

    // 获取element节点距离页面上、左边际的距离，以及节点的宽、高
    function getElementPosition(element) {
      var rect = getElementRect(element);
      return {
        x: rect.left,
        y: rect.top,
        width: rect.right - rect.left,
        height: rect.bottom - rect.top
      };
    }

    module.exports = getElementPosition;