# dangerousStyleValue::react-dom

## 概述

dangerousStyleValue(styleName,styleValue,component)，数值或数值型字符串样式值添加"px"作单位，其他转化为字符串输出。

## 源码

    'use strict';
    
    // CSSProperty.isUnitlessNumber[styleName]，判断是否无单位样式名的集合，有单位样式会对数值型字符串拼接px单位
    var CSSProperty = require('./CSSProperty');
    var warning = require('fbjs/lib/warning');
    
    var isUnitlessNumber = CSSProperty.isUnitlessNumber;
    var styleWarnings = {};
    
    // 数值或数值型字符串样式值添加"px"作单位，其他转化为字符串输出
    function dangerousStyleValue(name, value, component) {
      var isEmpty = value == null || typeof value === 'boolean' || value === '';
      if (isEmpty) {
        return '';
      }
    
      // 无单位样式直接转化为字符串
      var isNonNumeric = isNaN(value);// 数值或数值型字符串返回false
      if (isNonNumeric || value === 0 || isUnitlessNumber.hasOwnProperty(name) && isUnitlessNumber[name]) {
        return '' + value;
      }
    
      // 数值型且非0字符串警告未来将作为无单位样式值处理，不会自动拼接上"px"单位
      if (typeof value === 'string') {
        if (process.env.NODE_ENV !== 'production') {
          if (component && value !== '0') {
            var owner = component._currentElement._owner;// 父组件
            var ownerName = owner ? owner.getName() : null;
            if (ownerName && !styleWarnings[ownerName]) {
              styleWarnings[ownerName] = {};
            }
            var warned = false;
            if (ownerName) {
              var warnings = styleWarnings[ownerName];
              warned = warnings[name];
              if (!warned) {
                warnings[name] = true;
              }
            }
            if (!warned) {
              process.env.NODE_ENV !== 'production' ? 
                warning(false, 'a `%s` tag (owner: `%s`) was passed a numeric string value ' 
                  + 'for CSS property `%s` (value: `%s`) which will be treated ' 
                  + 'as a unitless number in a future version of React.', 
                  component._currentElement.type, ownerName || 'unknown', name, value) 
                : void 0;
            }
          }
        }
        value = value.trim();
      }
    
      // 数值或数值型字符串样式值添加"px"作单位
      return value + 'px';
    }
    
    module.exports = dangerousStyleValue;
    