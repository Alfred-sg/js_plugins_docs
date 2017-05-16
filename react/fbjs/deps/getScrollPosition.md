# getScrollPosition

getScrollPosition(scrollable)，获取滚动节点scrollable的偏移量，{x,y}形式输出。

    'use strict';

    var getDocumentScrollElement = require('./getDocumentScrollElement');
    var getUnboundedScrollPosition = require('./getUnboundedScrollPosition');

    // 获取滚动节点scrollable的偏移量，{x,y}形式输出
    function getScrollPosition(scrollable) {
      var documentScrollElement = getDocumentScrollElement();
      if (scrollable === window) {
        scrollable = documentScrollElement;
      }

      // scrollable为滚动节点，通过scrollLeft|scrollTop属性获取其偏移量，或window对象的pageOffset属性
      var scrollPosition = getUnboundedScrollPosition(scrollable);

      var viewport = scrollable === documentScrollElement ? document.documentElement : scrollable;

      // 计算滚动偏移量
      var xMax = scrollable.scrollWidth - viewport.clientWidth;
      var yMax = scrollable.scrollHeight - viewport.clientHeight;

      scrollPosition.x = Math.max(0, Math.min(scrollPosition.x, xMax));
      scrollPosition.y = Math.max(0, Math.min(scrollPosition.y, yMax));

      return scrollPosition;
    }

    module.exports = getScrollPosition;