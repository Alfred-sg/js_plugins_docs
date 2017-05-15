# containsNode

containsNode(outerNode, innerNode)，判断innerNode是否outerNode的子节点。

    'use strict';

    var isTextNode = require('./isTextNode');

    /**
     * 判断innerNode是否outerNode的子节点
     * @param  {node} outerNode 外层dom节点
     * @param  {node} innerNode 内层dom节点
     * @return {boolean}
     */
    function containsNode(outerNode, innerNode) {
      if (!outerNode || !innerNode) {
        return false;
      } else if (outerNode === innerNode) {
        return true;
      } else if (isTextNode(outerNode)) {
        return false;
      } else if (isTextNode(innerNode)) {
        return containsNode(outerNode, innerNode.parentNode);
      } else if ('contains' in outerNode) {
        return outerNode.contains(innerNode);
      } else if (outerNode.compareDocumentPosition) {
        return !!(outerNode.compareDocumentPosition(innerNode) & 16);
      } else {
        return false;
      }
    }

    module.exports = containsNode;