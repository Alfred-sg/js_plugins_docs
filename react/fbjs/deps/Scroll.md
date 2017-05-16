# Scroll

滚动偏移量操作。

    "use strict";

    // 判断element是否文档节点
    function _isViewportScrollElement(element, doc) {
      return !!doc && (element === doc.documentElement || element === doc.body);
    }

    var Scroll = {
      // 获取上下滚动偏移量
      getTop: function getTop(element) {
        var doc = element.ownerDocument;
        return _isViewportScrollElement(element, doc) ?
        doc.body.scrollTop || doc.documentElement.scrollTop : element.scrollTop;
      },

      // 设置上下滚动偏移量
      setTop: function setTop(element, newTop) {
        var doc = element.ownerDocument;
        if (_isViewportScrollElement(element, doc)) {
          doc.body.scrollTop = doc.documentElement.scrollTop = newTop;
        } else {
          element.scrollTop = newTop;
        }
      },

      // 获取左右滚动偏移量
      getLeft: function getLeft(element) {
        var doc = element.ownerDocument;
        return _isViewportScrollElement(element, doc) ? doc.body.scrollLeft || doc.documentElement.scrollLeft : element.scrollLeft;
      },

      // 设置左右滚动偏移量
      setLeft: function setLeft(element, newLeft) {
        var doc = element.ownerDocument;
        if (_isViewportScrollElement(element, doc)) {
          doc.body.scrollLeft = doc.documentElement.scrollLeft = newLeft;
        } else {
          element.scrollLeft = newLeft;
        }
      }
    };

    module.exports = Scroll;
