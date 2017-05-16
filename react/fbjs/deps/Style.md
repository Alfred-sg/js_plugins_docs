# Style

* Style.get(node,styleName)，获取node节点的styleName样式。
* Style.getScrollParent(node)，向上获取首个滚动的父元素，包含其自身。

    'use strict';

    var getStyleProperty = require('./getStyleProperty');

    // 判断是否滚动
    function _isNodeScrollable(element, name) {
      var overflow = Style.get(element, name);
      return overflow === 'auto' || overflow === 'scroll';
    }

    var Style = {
      // get(node,styleName)，获取node节点的styleName样式
      get: getStyleProperty,

      // 向上获取首个滚动的父元素，包含其自身
      getScrollParent: function getScrollParent(node) {
        if (!node) {
          return null;
        }
        while (node && node !== document.body) {
          if (_isNodeScrollable(node, 'overflow') || _isNodeScrollable(node, 'overflowY') || _isNodeScrollable(node, 'overflowX')) {
            return node;
          }
          node = node.parentNode;
        }
        return window;
      }

    };

    module.exports = Style;
