# createNodesFromMarkup

createNodesFromMarkup(markup, handleScript)，markup以html形式设置子节点，并返回子节点node对象集合nodeList；handleScript为真值允许设置script节点，并以函数形式处理script节点。

    'use strict';

    var ExecutionEnvironment = require('./ExecutionEnvironment');
    var createArrayFromMixed = require('./createArrayFromMixed');
    var getMarkupWrap = require('./getMarkupWrap');
    var invariant = require('./invariant');

    var dummyNode = ExecutionEnvironment.canUseDOM ? document.createElement('div') : null;

    var nodeNamePattern = /^\s*<(\w+)/;

    // 获取节点标签
    function getNodeName(markup) {
      var nodeNameMatch = markup.match(nodeNamePattern);
      return nodeNameMatch && nodeNameMatch[1].toLowerCase();
    }

    /**
     * 以html形式设置子节点，返回子节点node对象集合nodeList；handleScript为真值允许设置script节点
     * @param  {html} markup         html形式设置子节点
     * @param  {()=>{}} handleScript handleScript为真值允许设置script节点，函数形式处理script节点
     * @return {nodeList}            子节点集合
     */
    function createNodesFromMarkup(markup, handleScript) {
      var node = dummyNode;
      !!!dummyNode ? process.env.NODE_ENV !== 'production' ? 
        invariant(false, 'createNodesFromMarkup dummy not initialized') : invariant(false) : void 0;
      var nodeName = getNodeName(markup);

      var wrap = nodeName && getMarkupWrap(nodeName);
      if (wrap) {
        node.innerHTML = wrap[1] + markup + wrap[2];

        var wrapDepth = wrap[0];
        while (wrapDepth--) {
          node = node.lastChild;
        }
      } else {
        node.innerHTML = markup;
      }

      var scripts = node.getElementsByTagName('script');
      if (scripts.length) {
        !handleScript ? process.env.NODE_ENV !== 'production' ? 
          invariant(false, 'createNodesFromMarkup(...): Unexpected <script> element rendered.') 
          : invariant(false) : void 0;
        createArrayFromMixed(scripts).forEach(handleScript);
      }

      var nodes = Array.from(node.childNodes);

      // 已通过nodes获取子节点集合，移除dummyNode中的子节点
      while (node.lastChild) {
        node.removeChild(node.lastChild);
      }
      return nodes;
    }

    module.exports = createNodesFromMarkup;