# CSSProperty::react-dom

## 概述

* CSSProperty.isUnitlessNumber[styleName]，判断是否无单位样式名的集合，有单位样式会对数值型字符串拼接px单位。
* CSSProperty.shorthandPropertyExpansions[styleName]，判断是否复合样式，ie8下需对复合样式内的样式属性分别赋值。如background等复合样式，ie8不能对style.background赋值为""，而需要对style.background.backgroundColor等赋值为""。

## 源码

    'use strict';
    
    // 无单位样式，有单位样式会对数值型字符串拼接px单位
    var isUnitlessNumber = {
      animationIterationCount: true,
      borderImageOutset: true,
      borderImageSlice: true,
      borderImageWidth: true,
      boxFlex: true,
      boxFlexGroup: true,
      boxOrdinalGroup: true,
      columnCount: true,
      flex: true,
      flexGrow: true,
      flexPositive: true,
      flexShrink: true,
      flexNegative: true,
      flexOrder: true,
      gridRow: true,
      gridColumn: true,
      fontWeight: true,
      lineClamp: true,
      lineHeight: true,
      opacity: true,
      order: true,
      orphans: true,
      tabSize: true,
      widows: true,
      zIndex: true,
      zoom: true,
    
      // SVG相关样式
      fillOpacity: true,
      floodOpacity: true,
      stopOpacity: true,
      strokeDasharray: true,
      strokeDashoffset: true,
      strokeMiterlimit: true,
      strokeOpacity: true,
      strokeWidth: true
    };
    
    function prefixKey(prefix, key) {
      return prefix + key.charAt(0).toUpperCase() + key.substring(1);
    }
    
    var prefixes = ['Webkit', 'ms', 'Moz', 'O'];
    
    // 特定浏览器下的无单位样式
    Object.keys(isUnitlessNumber).forEach(function (prop) {
      prefixes.forEach(function (prefix) {
        isUnitlessNumber[prefixKey(prefix, prop)] = isUnitlessNumber[prop];
      });
    });
    
    // background等复合样式，ie8不能对style.background赋值为""，而需要对style.background.backgroundColor等赋值为""
    var shorthandPropertyExpansions = {
      background: {
        backgroundAttachment: true,
        backgroundColor: true,
        backgroundImage: true,
        backgroundPositionX: true,
        backgroundPositionY: true,
        backgroundRepeat: true
      },
      backgroundPosition: {
        backgroundPositionX: true,
        backgroundPositionY: true
      },
      border: {
        borderWidth: true,
        borderStyle: true,
        borderColor: true
      },
      borderBottom: {
        borderBottomWidth: true,
        borderBottomStyle: true,
        borderBottomColor: true
      },
      borderLeft: {
        borderLeftWidth: true,
        borderLeftStyle: true,
        borderLeftColor: true
      },
      borderRight: {
        borderRightWidth: true,
        borderRightStyle: true,
        borderRightColor: true
      },
      borderTop: {
        borderTopWidth: true,
        borderTopStyle: true,
        borderTopColor: true
      },
      font: {
        fontStyle: true,
        fontVariant: true,
        fontWeight: true,
        fontSize: true,
        lineHeight: true,
        fontFamily: true
      },
      outline: {
        outlineWidth: true,
        outlineStyle: true,
        outlineColor: true
      }
    };
    
    var CSSProperty = {
      // CSSProperty.isUnitlessNumber[styleName]，判断是否无单位样式名的集合，有单位样式会对数值型字符串拼接px单位
      isUnitlessNumber: isUnitlessNumber,
      // background等复合样式，ie8不能对style.background赋值为""，而需要对style.background.backgroundColor等赋值为""
      shorthandPropertyExpansions: shorthandPropertyExpansions
    };
    
    module.exports = CSSProperty;
