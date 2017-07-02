# AutoFocusUtils::react-dom

AutoFocusUtils.focusDOMComponent.call(inst)，ReactDomComponent实例对应节点获得焦点。

    'use strict';
    
    // ReactDOMComponentTree.getNodeFromInstance获取ReactDomComponent实例的对应节点
    var ReactDOMComponentTree = require('./ReactDOMComponentTree');
    
    var focusNode = require('fbjs/lib/focusNode');
    
    var AutoFocusUtils = {
      focusDOMComponent: function () {
        focusNode(ReactDOMComponentTree.getNodeFromInstance(this));
      }
    };
    
    // AutoFocusUtils.focusDOMComponent.call(inst)，ReactDomComponent实例对应节点获得焦点
    module.exports = AutoFocusUtils;

