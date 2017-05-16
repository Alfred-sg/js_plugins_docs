# getUnboundedScrollPosition

getUnboundedScrollPosition(scrollable)，获取scrollable滚动节点的偏移量或window对象的pageOffset属性，getScrollPosition模块中使用。

    'use strict';

    // 获取scrollable滚动节点的偏移量或window对象的pageOffset属性，getScrollPosition模块中使用
    function getUnboundedScrollPosition(scrollable) {
      if (scrollable === window) {
        return {
          x: window.pageXOffset || document.documentElement.scrollLeft,
          y: window.pageYOffset || document.documentElement.scrollTop
        };
      }
      return {
        x: scrollable.scrollLeft,
        y: scrollable.scrollTop
      };
    }

    module.exports = getUnboundedScrollPosition;