# CSSPropertyOperations::react-dom

## 概述

用于设置节点的样式。

* CSSPropertyOperations.createMarkupForStyles(styles,component)，将对象形式的样式styles拼接为字符串输出，component用于警告提示。以字符串拼接节点时使用。
* CSSPropertyOperations.setValueForStyles(node,styles,component)，将对象形式的样式styles转化为浏览器可接受的样式，然后设置为node节点的style样式，并使用ReactDebugTool记录样式更新操作。

## 源码

    'use strict';
    
    // CSSProperty.shorthandPropertyExpansions[styleName]，判断是否复合样式，ie8下需对复合样式内的样式属性分别赋值
    // 如background等复合样式，ie8不能对style.background赋值为""，而需要对style.background.backgroundColor等赋值为""
    var CSSProperty = require('./CSSProperty');
    
    // 本地开发环境调用ReactDebugTool调试函数库
    var ReactInstrumentation = require('./ReactInstrumentation');
    
    // dangerousStyleValue(styleName,styleValue,component)
    // 对数值或数值型字符串添加”px“单位处理，其他转化为字符串
    var dangerousStyleValue = require('./dangerousStyleValue');
    
    var ExecutionEnvironment = require('fbjs/lib/ExecutionEnvironment');
    
    // 字符串拼接的样式名转化为驼峰式输出，特别将"-ms-"起始的样式名转化成以"ms"起始
    var camelizeStyleName = require('fbjs/lib/camelizeStyleName');
    
    // 驼峰式样式名转化为连字符输出，如"msTransition"转化为"-ms-transition"
    // 将"backgroundColor"转化为"background-color"，"MozTransition"转化为"-moz-transition"
    var hyphenateStyleName = require('fbjs/lib/hyphenateStyleName');
    
    // 键值对缓存，回调函数以属性名为参数，返回值作为属性值；有属性值直接取值，不执行回调
    var memoizeStringOnly = require('fbjs/lib/memoizeStringOnly');
    
    var warning = require('fbjs/lib/warning');
    
    // 驼峰式样式名转化为连字符输出，并键值对缓存驼峰式样式名、连字符拼接的样式名
    var processStyleName = memoizeStringOnly(function (styleName) {
      return hyphenateStyleName(styleName);
    });
    
    // ie8下，是否不能对background等复合样式属性设为""，需要逐个设置backgroundColor样式为""
    var hasShorthandPropertyBug = false;
    
    // 特定浏览器下浮动样式属性名
    var styleFloatAccessor = 'cssFloat';
    if (ExecutionEnvironment.canUseDOM) {
      var tempStyle = document.createElement('div').style;
      try {
        tempStyle.font = '';
      } catch (e) {
        hasShorthandPropertyBug = true;
      }
        
      if (document.documentElement.style.cssFloat === undefined) {
        styleFloatAccessor = 'styleFloat';
      }
    }
    
    if (process.env.NODE_ENV !== 'production') {
      // 'msTransform'书写正确，"webkit|moz|o"首字母大写
      var badVendoredStyleNamePattern = /^(?:webkit|moz|o)[A-Z]/;
    
      var badStyleValueWithSemicolonPattern = /;\s*$/;
    
      var warnedStyleNames = {};
      var warnedStyleValues = {};
      var warnedForNaNValue = false;
    
      // 样式属性名含有"-"连字符号时作警告处理
      var warnHyphenatedStyleName = function (name, owner) {
        if (warnedStyleNames.hasOwnProperty(name) && warnedStyleNames[name]) {
          return;
        }
    
        warnedStyleNames[name] = true;
        process.env.NODE_ENV !== 'production' ? 
          warning(false, 'Unsupported style property %s. Did you mean %s?%s', 
            name, camelizeStyleName(name), checkRenderMessage(owner)) 
          : void 0;
      };
    
      // 特定浏览器样式属性名校验，webkit|moz|o首字母须为大写形式
      var warnBadVendoredStyleName = function (name, owner) {
        if (warnedStyleNames.hasOwnProperty(name) && warnedStyleNames[name]) {
          return;
        }
    
        warnedStyleNames[name] = true;
        process.env.NODE_ENV !== 'production' ? 
          warning(false, 'Unsupported vendor-prefixed style property %s. Did you mean %s?%s', 
            name, name.charAt(0).toUpperCase() + name.slice(1), checkRenderMessage(owner)) 
          : void 0;
      };
    
      // 样式值不能含有分号校验
      var warnStyleValueWithSemicolon = function (name, value, owner) {
        if (warnedStyleValues.hasOwnProperty(value) && warnedStyleValues[value]) {
          return;
        }
    
        warnedStyleValues[value] = true;
        process.env.NODE_ENV !== 'production' ? 
          warning(false, 'Style property values shouldn\'t contain a semicolon.%s ' 
            + 'Try "%s: %s" instead.', checkRenderMessage(owner), name, 
            value.replace(badStyleValueWithSemicolonPattern, '')) : 
          void 0;
      };
    
      // 样式值为NaN时警告处理
      var warnStyleValueIsNaN = function (name, value, owner) {
        if (warnedForNaNValue) {
          return;
        }
    
        warnedForNaNValue = true;
        process.env.NODE_ENV !== 'production' ? 
          warning(false, '`NaN` is an invalid value for the `%s` css style property.%s', 
            name, checkRenderMessage(owner)) 
          : void 0;
      };
    
      var checkRenderMessage = function (owner) {
        if (owner) {
          var name = owner.getName();
          if (name) {
            return ' Check the render method of `' + name + '`.';
          }
        }
        return '';
      };
    
      // 样式校验，样式名不能含有"-"连字符、webkit|moz|o首字母须为大写形式；样式值不能含有分号，及不能是NaN
      var warnValidStyle = function (name, value, component) {
        var owner;
        if (component) {
          owner = component._currentElement._owner;
        }
    
        // 样式属性含有"-"连字符号时作警告处理
        if (name.indexOf('-') > -1) {
          warnHyphenatedStyleName(name, owner);
    
        // 特定浏览器样式属性名校验，webkit|moz|o首字母须为大写形式
        } else if (badVendoredStyleNamePattern.test(name)) {
          warnBadVendoredStyleName(name, owner);
    
        // 样式值不能含有分号校验
        } else if (badStyleValueWithSemicolonPattern.test(value)) {
          warnStyleValueWithSemicolon(name, value, owner);
        }
    
        // 样式值为NaN时警告处理
        if (typeof value === 'number' && isNaN(value)) {
          warnStyleValueIsNaN(name, value, owner);
        }
      };
    }
    
    // 以字符串拼接节点的样式，或以node.style[styleName]=value设置节点的样式
    var CSSPropertyOperations = {
    
      // 将对象形式的样式styles拼接为字符串输出
      createMarkupForStyles: function (styles, component) {
        var serialized = '';
        for (var styleName in styles) {
          if (!styles.hasOwnProperty(styleName)) {
            continue;
          }
          var styleValue = styles[styleName];
          if (process.env.NODE_ENV !== 'production') {
            warnValidStyle(styleName, styleValue, component);
          }
          if (styleValue != null) {
            // processStyleName函数将驼峰式样式名转化为连字符输出
            serialized += processStyleName(styleName) + ':';
            // dangerousStyleValue函数对数值或数值型字符串添加”px“单位处理，其他转化为字符串
            serialized += dangerousStyleValue(styleName, styleValue, component) + ';';
          }
        }
        return serialized || null;
      },
    
      // 将styles对象转化为浏览器可接受的样式，通过node.style[styleName]=value以对象形式设置节点的样式
      //    并使用ReactDebugTool记录样式更新操作
      setValueForStyles: function (node, styles, component) {
        if (process.env.NODE_ENV !== 'production') {
          ReactInstrumentation.debugTool.onHostOperation({
            instanceID: component._debugID,
            type: 'update styles',
            payload: styles
          });
        }
    
        var style = node.style;
        for (var styleName in styles) {
          if (!styles.hasOwnProperty(styleName)) {
            continue;
          }
          if (process.env.NODE_ENV !== 'production') {
            // warnValidStyle样式校验，样式名不能含有"-"连字符、webkit|moz|o首字母须为大写形式；样式值不能含有分号，及不能是NaN
            warnValidStyle(styleName, styles[styleName], component);
          }
    
          // dangerousStyleValue函数对数值或数值型字符串添加”px“单位处理，其他转化为字符串
          var styleValue = dangerousStyleValue(styleName, styles[styleName], component);
    
          // 置换为特定浏览器下浮动样式属性名，'cssFloat'或'styleFloat'
          if (styleName === 'float' || styleName === 'cssFloat') {
            styleName = styleFloatAccessor;
          }
    
          if (styleValue) {
            style[styleName] = styleValue;
          } else {
            // ie8下，对background等复合样式属性，逐个设置backgroundColor样式为""
            var expansion = hasShorthandPropertyBug && CSSProperty.shorthandPropertyExpansions[styleName];
            if (expansion) {
              for (var individualStyleName in expansion) {
                style[individualStyleName] = '';
              }
            } else {
              style[styleName] = '';
            }
          }
        }
      }
    
    };
    
    module.exports = CSSPropertyOperations;
