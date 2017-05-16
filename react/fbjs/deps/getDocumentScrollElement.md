# getDocumentScrollElement

getDocumentScrollElement()，为滚动取值获取文档节点，getScrollPosition函数中使用。

    'use strict';

    var isWebkit = typeof navigator !== 'undefined' && navigator.userAgent.indexOf('AppleWebKit') > -1;

    // 为滚动状态获取文档节点，getScrollPosition模块中使用
    function getDocumentScrollElement(doc) {
      doc = doc || document;
      return !isWebkit && doc.compatMode === 'CSS1Compat' ? doc.documentElement : doc.body;
    }

    module.exports = getDocumentScrollElement;