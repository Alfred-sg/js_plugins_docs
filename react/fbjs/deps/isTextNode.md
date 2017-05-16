# isTextNode

isTextNode(node)，基于isNode函数，判断node是否文本节点。

    'use strict';

    var isNode = require('./isNode');

    function isTextNode(object) {
      return isNode(object) && object.nodeType == 3;
    }

    module.exports = isTextNode;