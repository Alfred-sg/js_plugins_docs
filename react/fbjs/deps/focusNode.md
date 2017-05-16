# focusNode

focusNode(node)，使node节点获得焦点。

    'use strict';

    function focusNode(node) {
      try {
        node.focus();
      } catch (e) {}
    }

    module.exports = focusNode;