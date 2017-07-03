# DOMPropertyOperations::react-dom

## 概述

节点属性操作。以拼接字符串方式生成react元素时，属性也使用拼接字符串形式设置；dom节点操作生成react元素时，属性也使用浏览器原生dom节点方式设置。

* DOMPropertyOperations.createMarkupForID(id)，拼接字符串形式设置节点的'data-reactid'属性。
* DOMPropertyOperations.setAttributeForID(node,id)，node.setAttribute形式设置节点的'data-reactid'属性。
* DOMPropertyOperations.createMarkupForRoot()，拼接字符串形式设置节点的'data-reactroot'属性。
* DOMPropertyOperations.setAttributeForRoot(node)，node.setAttribute形式设置节点的'data-reactroot'属性。
* DOMPropertyOperations.createMarkupForProperty(name,value)，拼接字符串形式设置节点的property类属性。
* DOMPropertyOperations.createMarkupForCustomAttribute(name,value)，拼接字符串形式设置节点的attribute类自定义属性。
* DOMPropertyOperations.setValueForProperty(node,name,value)，设置property类节点属性。当propertyInfo.mutationMethod存在时，使用该方法设置节点的property属性；当propertyInfo.mustUseProperty为真值时，使用node[propertyInfo.propertyName]=value设置节点属性；当属性名通过DOMProperty.isCustomAttribute校验，使用DOMPropertyOperations.setValueForAttribute设置节点属性；其余，使用setAttribute或setAttributeNS(属性含命名空间时启用)方法设置节点属性。开发环境使用ReactDebugTool相关缓存记录节点属性操作。
* DOMPropertyOperations.setValueForAttribute(node,name,value)，设置attribute类节点属性。若value非null，使用node.setAttribute设置节点属性；value为null，使用node.removeAttribute移除节点属性。开发环境使用ReactDebugTool相关缓存记录节点属性操作。
* DOMPropertyOperations.deleteValueForAttribute(node,name)，移除attribute类节点属性。使用node.removeAttribute移除节点属性name。开发环境使用ReactDebugTool相关缓存记录节点属性操作。
* DOMPropertyOperations.deleteValueForProperty(node,name)，移除property类节点属性。当propertyInfo.mutationMethod存在时，优先使用该方法移除节点属性；当propertyInfo.mustUseProperty为真值，使用node[propertyInfo.propertyName]移除节点属性；其余，使用node.removeAttribute方法方法移除节点属性。开发环境使用ReactDebugTool相关缓存记录节点属性操作。

## 源码

    'use strict';
    
    // DOMProperty.properties = {propName: {attributeName,attributeNamespace,propertyName,
    //    mutationMethod,mustUseProperty,hasBooleanValue,hasNumericValue,hasPositiveNumericValue,
    //    hasOverloadedBooleanValue}}
    //    限制节点属性的设置方式、属性类型、可接受的属性值等
    // DOMProperty.isCustomAttribute，校验是否react内置的自定义属性
    //    如HTMLDOMPropertyConfig模块设定以data-或aria-起始的属性
    var DOMProperty = require('./DOMProperty');
    
    // ReactDOMComponentTree.getInstanceFromNode(node)
    //  获取与node对应的ReactDomComponent或ReactDOMTextComponent实例
    var ReactDOMComponentTree = require('./ReactDOMComponentTree');
    
    // 本地开发环境调用ReactDebugTool调试函数库
    var ReactInstrumentation = require('./ReactInstrumentation');
    
    // 布尔、数值型转化为字符串，字符串经html转码，处理["'&<>]；外加引号包裹
    var quoteAttributeValueForBrowser = require('./quoteAttributeValueForBrowser');
    
    var warning = require('fbjs/lib/warning');
    
    // 用于校验属性名，英文字母a-zA-Z、":"、"-"、数字、部分Unicode字符形式
    var VALID_ATTRIBUTE_NAME_REGEX = 
      new RegExp('^[' + DOMProperty.ATTRIBUTE_NAME_START_CHAR + '][' + DOMProperty.ATTRIBUTE_NAME_CHAR + ']*$');
    
    var illegalAttributeNameCache = {};// 缓存校验失败的属性名
    var validatedAttributeNameCache = {};// 缓存校验成功的属性名
    
    // 校验属性名，英文字母a-zA-Z、":"、"-"、数字、部分Unicode字符形式
    function isAttributeNameSafe(attributeName) {
      if (validatedAttributeNameCache.hasOwnProperty(attributeName)) {
        return true;
      }
      if (illegalAttributeNameCache.hasOwnProperty(attributeName)) {
        return false;
      }
      if (VALID_ATTRIBUTE_NAME_REGEX.test(attributeName)) {
        validatedAttributeNameCache[attributeName] = true;
        return true;
      }
      illegalAttributeNameCache[attributeName] = true;
      process.env.NODE_ENV !== 'production' ? 
        warning(false, 'Invalid attribute name: `%s`', attributeName) : void 0;
      return false;
    }
    
    // 属性值约定的布尔型真值校验(否值忽略)、数值型校验、正数校验
    function shouldIgnoreValue(propertyInfo, value) {
      return value == null || propertyInfo.hasBooleanValue && !value || 
        propertyInfo.hasNumericValue && isNaN(value) || 
        propertyInfo.hasPositiveNumericValue && value < 1 || 
        propertyInfo.hasOverloadedBooleanValue && value === false;
    }
    
    // 节点属性操作，添加或移除属性
    var DOMPropertyOperations = {
    
      // 以字符串形式设置节点的'data-reactid'属性及值；拼接字符串的方式创建节点时使用
      createMarkupForID: function (id) {
        return DOMProperty.ID_ATTRIBUTE_NAME + '=' + quoteAttributeValueForBrowser(id);
      },
    
      // 设置节点的'data-reactid'的属性；document.createElement方式创建节点时使用
      setAttributeForID: function (node, id) {
        node.setAttribute(DOMProperty.ID_ATTRIBUTE_NAME, id);
      },
    
      // 以字符串形式设置节点的'data-reactroot'属性及值；拼接字符串的方式创建节点时使用
      createMarkupForRoot: function () {
        return DOMProperty.ROOT_ATTRIBUTE_NAME + '=""';
      },
    
      // 设置节点的'data-reactroot'属性；document.createElement方式创建节点时使用
      setAttributeForRoot: function (node) {
        node.setAttribute(DOMProperty.ROOT_ATTRIBUTE_NAME, '');
      },
    
      // 以拼接字符串方式设置property类属性
      createMarkupForProperty: function (name, value) {
        var propertyInfo = DOMProperty.properties.hasOwnProperty(name) ? DOMProperty.properties[name] : null;
        if (propertyInfo) {
    
          // 以propertyInfo约定的属性值类型忽略某些属性值value，如properties.hasBooleanValue设置会忽略属性值为否值的value
          if (shouldIgnoreValue(propertyInfo, value)) {
            return '';
          }
          var attributeName = propertyInfo.attributeName;
          if (propertyInfo.hasBooleanValue || propertyInfo.hasOverloadedBooleanValue && value === true) {
            return attributeName + '=""';
          }
          return attributeName + '=' + quoteAttributeValueForBrowser(value);
        } else if (DOMProperty.isCustomAttribute(name)) {
          if (value == null) {
            return '';
          }
          return name + '=' + quoteAttributeValueForBrowser(value);
        }
        return null;
      },
    
      // 以拼接字符串方式设置attribute类自定义属性
      createMarkupForCustomAttribute: function (name, value) {
        if (!isAttributeNameSafe(name) || value == null) {
          return '';
        }
        return name + '=' + quoteAttributeValueForBrowser(value);
      },
    
      // 设置property类节点属性
      // 当propertyInfo.mutationMethod存在时，使用该方法设置节点的property属性
      // 当propertyInfo.mustUseProperty为真值时，使用node[propertyInfo.propertyName]=value设置节点属性
      // 当属性名通过DOMProperty.isCustomAttribute校验，使用DOMPropertyOperations.setValueForAttribute设置节点属性
      // 其余，使用setAttribute或setAttributeNS(属性含命名空间时启用)方法设置节点属性
      setValueForProperty: function (node, name, value) {
        var propertyInfo = DOMProperty.properties.hasOwnProperty(name) ? DOMProperty.properties[name] : null;
        if (propertyInfo) {
    
          // 以节点属性插件的DOMMutationMethods配置方法添加节点属性
          var mutationMethod = propertyInfo.mutationMethod;
          if (mutationMethod) {
            mutationMethod(node, value);
    
          // 以propertyInfo约定的属性值类型忽略某些属性值value，如properties.hasBooleanValue设置会忽略属性值为否值的value
          } else if (shouldIgnoreValue(propertyInfo, value)) {
            this.deleteValueForProperty(node, name);
            return;
    
          // propertyInfo.mustUseProperty为真值时，不用setAttribute方法设置节点属性
          // 处理ie8、9setAttribute方法将属性值转换成字符串`[object]`的兼容性问题
          } else if (propertyInfo.mustUseProperty) {
            node[propertyInfo.propertyName] = value;
          } else {
            var attributeName = propertyInfo.attributeName;
            var namespace = propertyInfo.attributeNamespace;
    
            // node.setAttributeNS(namespace,attr,value)添加特定命名空间的属性
            if (namespace) {
              node.setAttributeNS(namespace, attributeName, '' + value);
            // 布尔型往节点注入属性名即可
            } else if (propertyInfo.hasBooleanValue || propertyInfo.hasOverloadedBooleanValue && value === true) {
              node.setAttribute(attributeName, '');
            } else {
              node.setAttribute(attributeName, '' + value);
            }
          }
        // 对通过DOMProperty.isCustomAttribute校验的属性，根据value值移除或添加属性
        } else if (DOMProperty.isCustomAttribute(name)) {
          DOMPropertyOperations.setValueForAttribute(node, name, value);
          return;
        }
    
        if (process.env.NODE_ENV !== 'production') {
          var payload = {};
          payload[name] = value;
          ReactInstrumentation.debugTool.onHostOperation({
            instanceID: ReactDOMComponentTree.getInstanceFromNode(node)._debugID,
            type: 'update attribute',
            payload: payload
          });
        }
      },
    
      // 设置attribute类节点属性
      // 若value非null，使用node.setAttribute设置节点属性；value为null，使用node.removeAttribute移除节点属性
      setValueForAttribute: function (node, name, value) {
        if (!isAttributeNameSafe(name)) {
          return;
        }
        if (value == null) {
          node.removeAttribute(name);
        } else {
          node.setAttribute(name, '' + value);
        }
    
        if (process.env.NODE_ENV !== 'production') {
          var payload = {};
          payload[name] = value;
          ReactInstrumentation.debugTool.onHostOperation({
            instanceID: ReactDOMComponentTree.getInstanceFromNode(node)._debugID,
            type: 'update attribute',
            payload: payload
          });
        }
      },
    
      // 移除attribute类节点属性
      // 使用node.removeAttribute移除节点属性name，开发环境使用ReactDebugTool相关缓存记录节点属性操作
      deleteValueForAttribute: function (node, name) {
        node.removeAttribute(name);
        if (process.env.NODE_ENV !== 'production') {
          ReactInstrumentation.debugTool.onHostOperation({
            instanceID: ReactDOMComponentTree.getInstanceFromNode(node)._debugID,
            type: 'remove attribute',
            payload: name
          });
        }
      },
    
      // 移除property类节点属性
      // 当propertyInfo.mutationMethod存在时，优先使用该方法移除节点属性
      // 当propertyInfo.mustUseProperty为真值，使用node[propertyInfo.propertyName]移除节点属性
      // 其余，使用node.removeAttribute方法方法移除节点属性
      // 开发环境使用ReactDebugTool相关缓存记录节点属性操作
      deleteValueForProperty: function (node, name) {
        var propertyInfo = DOMProperty.properties.hasOwnProperty(name) ? DOMProperty.properties[name] : null;
        if (propertyInfo) {
          var mutationMethod = propertyInfo.mutationMethod;
          if (mutationMethod) {
            mutationMethod(node, undefined);
          } else if (propertyInfo.mustUseProperty) {
            var propName = propertyInfo.propertyName;
            if (propertyInfo.hasBooleanValue) {
              node[propName] = false;
            } else {
              node[propName] = '';
            }
          } else {
            node.removeAttribute(propertyInfo.attributeName);
          }
        } else if (DOMProperty.isCustomAttribute(name)) {
          node.removeAttribute(name);
        }
    
        if (process.env.NODE_ENV !== 'production') {
          ReactInstrumentation.debugTool.onHostOperation({
            instanceID: ReactDOMComponentTree.getInstanceFromNode(node)._debugID,
            type: 'remove attribute',
            payload: name
          });
        }
      }
    
    };
    
    module.exports = DOMPropertyOperations;
