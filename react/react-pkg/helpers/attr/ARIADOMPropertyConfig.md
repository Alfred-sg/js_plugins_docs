# ARIADOMPropertyConfig::react-dom

## 概述

属性配置管理对象，可以含有如下参数: 
isCustomAttribute: 函数，判断是否可接受的自定义属性，返回真值将添置节点的属性。如HTMLDOMPropertyConfig模块的"data-*"、"aria-*"属性。
Properties: 约定node[propName]属性设置方式及属性类型。如Properties[propName]为MUST_USE_PROPERTY即0x1时，须以node[propName]=value方式添加属性；如Properties[propName]为HAS_BOOLEAN_VALUE即0x4时，否值将不会添加为节点的属性。
DOMAttributeNames: react约定属性名到浏览器节点属性名的映射，如{className:"class"}。
DOMAttributeNamespaces: 约定属性命名空间。如DOMAttributeNamespaces[propName]=namespace时，需用node.setAttributeNS(propName,value)方法设置属性。
DOMPropertyNames: node[propName]=value方式设置节点属性时，用以提供propName值。
DOMMutationMethods: 设定节点属性的方法集合，如{propName:(node,value)=>{}}。

react内置自定义aria-*属性。

## 源码

    'use strict';
    
    var ARIADOMPropertyConfig = {
      // dom节点可设置的属性，作为DOMProperty.Properties的键
      Properties: {
        // Global States and Properties
        'aria-current': 0, // state // 0，设置属性时，不限制数据形式
        'aria-details': 0,
        'aria-disabled': 0, // state
        'aria-hidden': 0, // state
        'aria-invalid': 0, // state
        'aria-keyshortcuts': 0,
        'aria-label': 0,
        'aria-roledescription': 0,
        // Widget Attributes
        'aria-autocomplete': 0,
        'aria-checked': 0,
        'aria-expanded': 0,
        'aria-haspopup': 0,
        'aria-level': 0,
        'aria-modal': 0,
        'aria-multiline': 0,
        'aria-multiselectable': 0,
        'aria-orientation': 0,
        'aria-placeholder': 0,
        'aria-pressed': 0,
        'aria-readonly': 0,
        'aria-required': 0,
        'aria-selected': 0,
        'aria-sort': 0,
        'aria-valuemax': 0,
        'aria-valuemin': 0,
        'aria-valuenow': 0,
        'aria-valuetext': 0,
        // Live Region Attributes
        'aria-atomic': 0,
        'aria-busy': 0,
        'aria-live': 0,
        'aria-relevant': 0,
        // Drag-and-Drop Attributes
        'aria-dropeffect': 0,
        'aria-grabbed': 0,
        // Relationship Attributes
        'aria-activedescendant': 0,
        'aria-colcount': 0,
        'aria-colindex': 0,
        'aria-colspan': 0,
        'aria-controls': 0,
        'aria-describedby': 0,
        'aria-errormessage': 0,
        'aria-flowto': 0,
        'aria-labelledby': 0,
        'aria-owns': 0,
        'aria-posinset': 0,
        'aria-rowcount': 0,
        'aria-rowindex': 0,
        'aria-rowspan': 0,
        'aria-setsize': 0
      },
      DOMAttributeNames: {},
      DOMPropertyNames: {}
    };
    
    module.exports = ARIADOMPropertyConfig;
    