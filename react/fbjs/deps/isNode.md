# isNode

isNode(node)，判断node是否dom节点。

    'use strict';

    // 判断是否dom节点
    function isNode(object) {
      return !!(object && (typeof Node === 'function' ? 
        object instanceof Node : typeof object === 'object' && typeof object.nodeType === 'number' && 
        typeof object.nodeName === 'string'));
    }

    module.exports = isNode;