# DOMProperty::react-dom

## 概述

用于校验节点属性，设定节点属性的设置方式、属性类型、可接受的属性值等，以及校验是否react内置自定义属性。

* DOMProperty.ID_ATTRIBUTE_NAME = 'data-reactid'
* DOMProperty.ROOT_ATTRIBUTE_NAME = 'data-reactroot'
* DOMProperty.ATTRIBUTE_NAME_START_CHAR、DOMProperty.ATTRIBUTE_NAME_CHAR，用于校验属性名起始字符串和属性名。
* DOMProperty.properties = {propName: {attributeName,attributeNamespace,propertyName,mutationMethod,mustUseProperty,hasBooleanValue,hasNumericValue,hasPositiveNumericValue,hasOverloadedBooleanValue}}，限制节点属性的设置方式、属性类型、可接受的属性值等。
* DOMProperty._isCustomAttributeFunctions = [(propertyName)=>{...;return [true|false];}]，自定义属性校验函数数组。
* DOMProperty.isCustomAttribute，校验是否react内置的自定义属性，如HTMLDOMPropertyConfig模块设定以data-或aria-起始的属性。
* DOMProperty.injection.injectDOMPropertyConfig(domPropertyConfig)，通过ReactDefaultInjection模块加载节点属性配置管理对象，如ARIADOMPropertyConfig、HTMLDOMPropertyConfig、SVGDOMPropertyConfig，最终为当前模块的DOMProperty.properties、DOMProperty._isCustomAttributeFunctions注入内容。

## 源码

    'use strict';
    
    // reactProdInvariant(code)，线上环境报错，提示facebook文档
    var _prodInvariant = require('./reactProdInvariant');
    
    var invariant = require('fbjs/lib/invariant');
    
    // 获取是否以node[propName]=value方式添加属性，或属性值处理方式
    function checkMask(value, bitmask) {
      return (value & bitmask) === bitmask;
    }
    
    var DOMPropertyInjection = {
      MUST_USE_PROPERTY: 0x1,// 0x 16位 以node[propName]=value方式添加属性
      HAS_BOOLEAN_VALUE: 0x4,// 过滤否值
      HAS_NUMERIC_VALUE: 0x8,// 过滤非数值型
      HAS_POSITIVE_NUMERIC_VALUE: 0x10 | 0x8,// 过滤非正数型
      HAS_OVERLOADED_BOOLEAN_VALUE: 0x20,// 过滤false
    
      // 加载节点属性配置管理对象，为DOMProperty.properties、DOMProperty._isCustomAttributeFunctions注入内容
      // 
      // 参数domPropertyConfig，节点属性配置管理对象，含有如下属性:
      // isCustomAttribute: 函数，判断是否可接受的自定义属性，返回真值将添置节点的属性
      //    如HTMLDOMPropertyConfig模块的"data-*"、"aria-*"属性
      // Properties: 约定node[propName]属性设置方式及属性类型
      //    如Properties[propName]为MUST_USE_PROPERTY即0x1时，须以node[propName]=value方式添加属性
      //    如Properties[propName]为HAS_BOOLEAN_VALUE即0x4时，否值将不会添加为节点的属性
      // DOMAttributeNames: react约定属性名到浏览器节点属性名的映射，如{className:"class"}
      // DOMAttributeNamespaces: 约定属性命名空间
      //    如DOMAttributeNamespaces[propName]=namespace时，需用node.setAttributeNS(propName,value)方法设置属性
      // DOMPropertyNames: node[propName]=value方式设置节点属性时，用以提供propName值
      // DOMMutationMethods: 设定节点属性的方法集合，如{propName:(node,value)=>{}}
      // 
      // DOMProperty.properties={propName:propertyInfo}
      // propertyInfo约定属性的类型、属性的命名空间、属性的设置方式等，含有如下属性: 
      //    attributeName: 拼接字符串方式设置节点属性时，作为节点的属性名
      //    attributeNamespace: 属性的命名空间，需用node.setAttributeNS(propName,value)方法设置属性
      //    propertyName: node[propName]=value方式设置节点属性时，用以提供propName值；
      //        默认为propertyInfo.attributeName；自定义为domPropertyConfig.DOMPropertyNames[propName]
      //    mutationMethod: 设置属性值的方法，(node,value)=>{}形式，优先级最高
      //    mustUseProperty: 是否以node[propertyInfo[propertyName]]=value设置节点属性，而非setAttribute方法
      //    hasBooleanValue: 真值，不接受否值作为节点的属性
      //    hasNumericValue: 真值，只接受数值型或可转化成数值型的字符串作为节点的属性
      //    hasPositiveNumericValue: 真值，只接受正数作为节点的属性
      //    hasOverloadedBooleanValue: 真值，不接受false作为节点的属性
      // DOMProperty._isCustomAttributeFunctions=[propertyName=>[true|false]]，自定义属性的校验函数数组
      injectDOMPropertyConfig: function (domPropertyConfig) {
        var Injection = DOMPropertyInjection;
    
        var Properties = domPropertyConfig.Properties || {};
        var DOMAttributeNamespaces = domPropertyConfig.DOMAttributeNamespaces || {};
        var DOMAttributeNames = domPropertyConfig.DOMAttributeNames || {};
        var DOMPropertyNames = domPropertyConfig.DOMPropertyNames || {};
        var DOMMutationMethods = domPropertyConfig.DOMMutationMethods || {};
    
        // 自定义属性校验，通过则将添加到节点的相应属性上，如HTMLDOMPropertyConfig模块的data-、aria-
        if (domPropertyConfig.isCustomAttribute) {
          DOMProperty._isCustomAttributeFunctions.push(domPropertyConfig.isCustomAttribute);
        }
    
        for (var propName in Properties) {
          // 同名属性不能加载两次，也即各节点属性插件的属性名不能相冲
          !!DOMProperty.properties.hasOwnProperty(propName) ? 
            process.env.NODE_ENV !== 'production' ? 
              invariant(false, 'injectDOMPropertyConfig(...):' 
                + ' You\'re trying to inject DOM property \'%s\' which has already been injected.' 
                + ' You may be accidentally injecting the same DOM property config twice,' 
                + ' or you may be injecting two configs that have conflicting property names.', propName) 
              : _prodInvariant('48', propName) : void 0;
    
          var lowerCased = propName.toLowerCase();
    
          var propConfig = Properties[propName];
    
          var propertyInfo = {
            // 拼接字符串方式添加节点属性时，作为属性名
            attributeName: lowerCased,
    
            // 属性的命名空间，有值时需用node.setAttributeNS(namespace,attr,value)添加属性；初始化为null
            attributeNamespace: null,
    
            // propertyInfo.mustUseProperty为真值时，需用node[propertyInfo[propertyName]]设置节点属性
            propertyName: propName,
    
            // 设置属性值的方法，(node,value)=>{}形式，最高；初始化为null
            mutationMethod: null,
    
            // 为真值时，以node[propertyInfo[propertyName]]设置节点的属性，而非setAttribute方法
            // 处理ie8、9setAttribute方法将属性值转化为字符串`[object]`的兼容性问题
            mustUseProperty: checkMask(propConfig, Injection.MUST_USE_PROPERTY),
    
            // 用于设定节点属性值的类型，如hasNumericValue属性为真值，属性值得类型即为数值型
            hasBooleanValue: checkMask(propConfig, Injection.HAS_BOOLEAN_VALUE),
            hasNumericValue: checkMask(propConfig, Injection.HAS_NUMERIC_VALUE),
            hasPositiveNumericValue: checkMask(propConfig, Injection.HAS_POSITIVE_NUMERIC_VALUE),
            hasOverloadedBooleanValue: checkMask(propConfig, Injection.HAS_OVERLOADED_BOOLEAN_VALUE)
          };
    
          // 属性值的类型设定相冲，如不能既是布尔型，又是数值型
          !(propertyInfo.hasBooleanValue + propertyInfo.hasNumericValue + 
            propertyInfo.hasOverloadedBooleanValue <= 1) ? 
              process.env.NODE_ENV !== 'production' ? 
                invariant(false, 'DOMProperty: Value can be one of boolean, overloaded boolean,' 
                  + ' or numeric value, but not a combination: %s', propName) : 
                _prodInvariant('50', propName) : 
              void 0;
    
          if (process.env.NODE_ENV !== 'production') {
            DOMProperty.getPossibleStandardName[lowerCased] = propName;
          }
    
          // 节点属性插件的DOMAttributeNames属性是react属性名到浏览器节点属性名的映射，如className映射为class
          // 以字符串形式拼接属性名及其值时使用
          if (DOMAttributeNames.hasOwnProperty(propName)) {
            var attributeName = DOMAttributeNames[propName];
            propertyInfo.attributeName = attributeName;
            if (process.env.NODE_ENV !== 'production') {
              DOMProperty.getPossibleStandardName[attributeName] = propName;
            }
          }
    
          // 提取节点属性插件DOMAttributeNamespaces的命名空间
          if (DOMAttributeNamespaces.hasOwnProperty(propName)) {
            propertyInfo.attributeNamespace = DOMAttributeNamespaces[propName];
          }
    
          // DOMPropertyNames存放propertyInfo.mustUseProperty为真值时可添加的属性名集合
          if (DOMPropertyNames.hasOwnProperty(propName)) {
            propertyInfo.propertyName = DOMPropertyNames[propName];
          }
    
          // propertyInfo.mutationMethod设置属性值的方法，(node,value)=>{}形式
          if (DOMMutationMethods.hasOwnProperty(propName)) {
            propertyInfo.mutationMethod = DOMMutationMethods[propName];
          }
    
          DOMProperty.properties[propName] = propertyInfo;
        }
      }
    };
    
    var ATTRIBUTE_NAME_START_CHAR = ':A-Z_a-z\\u00C0-\\u00D6\\u00D8-\\u00F6\\u00F8-\\u02FF\\u0370-\\u037D\\u037F-\\u1FFF\\u200C-\\u200D\\u2070-\\u218F\\u2C00-\\u2FEF\\u3001-\\uD7FF\\uF900-\\uFDCF\\uFDF0-\\uFFFD';
    
    var DOMProperty = {
    
      ID_ATTRIBUTE_NAME: 'data-reactid',
      ROOT_ATTRIBUTE_NAME: 'data-reactroot',
    
      // 用于校验属性名
      ATTRIBUTE_NAME_START_CHAR: ATTRIBUTE_NAME_START_CHAR,
      ATTRIBUTE_NAME_CHAR: ATTRIBUTE_NAME_START_CHAR + '\\-.0-9\\u00B7\\u0300-\\u036F\\u203F-\\u2040',
    
      // {propName:propertyInfo}，限制节点属性的类型、设置方式、可接受的值等
      // 
      // propertyInfo包含如下属性：
      // attributeName: 拼接字符串方式设置节点属性时，作为节点的属性名
      // attributeNamespace: 属性的命名空间，需用node.setAttributeNS(propName,value)方法设置属性
      // propertyName: node[propName]=value方式设置节点属性时，用以提供propName值；
      //    默认为propertyInfo.attributeName；自定义为domPropertyConfig.DOMPropertyNames[propName]
      // mutationMethod: 设置属性值的方法，(node,value)=>{}形式，优先级最高
      // mustUseProperty: 是否以node[propertyInfo[propertyName]]=value设置节点属性，而非setAttribute方法
      // hasBooleanValue: 真值，不接受否值作为节点的属性
      // hasNumericValue: 真值，只接受数值型或可转化成数值型的字符串作为节点的属性
      // hasPositiveNumericValue: 真值，只接受正数作为节点的属性
      // hasOverloadedBooleanValue: 真值，不接受false作为节点的属性
      properties: {},
    
      // 存储小写形式的节点属性到react节点属性的映射，如{class:"className"}等；调试用
      getPossibleStandardName: process.env.NODE_ENV !== 'production' ? { autofocus: 'autoFocus' } : null,
    
      // [(propertyName)=>{...;return [true|false];}]，自定义属性校验函数数组
      _isCustomAttributeFunctions: [],
    
      // 校验是否react内置的自定义属性，如HTMLDOMPropertyConfig模块设定以data-或aria-起始的属性
      isCustomAttribute: function (attributeName) {
        for (var i = 0; i < DOMProperty._isCustomAttributeFunctions.length; i++) {
          var isCustomAttributeFn = DOMProperty._isCustomAttributeFunctions[i];
          if (isCustomAttributeFn(attributeName)) {
            return true;
          }
        }
        return false;
      },
    
      // 通过ReactDefaultInjection模块加载节点属性配置管理对象，如ARIADOMPropertyConfig、HTMLDOMPropertyConfig、SVGDOMPropertyConfig
      // 最终为当前模块的DOMProperty.properties、DOMProperty._isCustomAttributeFunctions注入内容
      injection: DOMPropertyInjection
    };
    
    module.exports = DOMProperty;
