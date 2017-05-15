# CSSCore

样式操作。

    'use strict';

    var invariant = require('./invariant');

    // 由element的顶层节点root判断element是否匹配selector选择器
    function matchesSelector_SLOW(element, selector) {
      var root = element;
      while (root.parentNode) {
        root = root.parentNode;
      }

      var all = root.querySelectorAll(selector);
      return Array.prototype.indexOf.call(all, element) !== -1;
    }

    var CSSCore = {

      /**
       * 通过element.classList.add方法或者重新赋值element.className，添加单个样式className
       * @param {DOMElement}  dom元素
       * @param {string}      待删除的样式；一次只能添加一个样式
       * @return {DOMElement} 返回添加样式后的dom元素
       */
      addClass: function addClass(element, className) {
        // 一次只能删除一个样式
        !!/\s/.test(className) ? process.env.NODE_ENV !== 'production' ? 
          invariant(false, 'CSSCore.addClass takes only a single class name. "%s" contains ' 
            + 'multiple classes.', className) : invariant(false) : void 0;

        if (className) {
          if (element.classList) {
            element.classList.add(className);
          } else if (!CSSCore.hasClass(element, className)) {
            element.className = element.className + ' ' + className;
          }
        }
        return element;
      },

      /**
       * 通过element.classList.remove方法或者重新赋值element.className，移除单个样式className
       * @param {DOMElement}  dom元素
       * @param {string}      待删除的样式；一次只能删除一个样式
       * @return {DOMElement} 返回删除样式后的dom元素
       */
      removeClass: function removeClass(element, className) {
        !!/\s/.test(className) ? process.env.NODE_ENV !== 'production' ? 
          invariant(false, 'CSSCore.removeClass takes only a single class name. "%s" contains ' 
            + 'multiple classes.', className) : invariant(false) : void 0;

        if (className) {
          if (element.classList) {
            element.classList.remove(className);
          } else if (CSSCore.hasClass(element, className)) {
            element.className = element.className.replace(new RegExp('(^|\\s)' + className + '(?:\\s|$)', 'g'), '$1').replace(/\s+/g, ' ') // multiple spaces to one
            .replace(/^\s*|\s*$/g, ''); // trim the ends
          }
        }
        return element;
      },

      /**
       * 根据bool，添加或移除element的样式className
       * @param {DOMElement}  dom元素
       * @param {string}      待添加或删除的样式；一次只能添加或删除一个样式
       * @param {*} bool      真值添加样式，否值移除样式
       * @return {DOMElement} 返回添加或删除样式后的dom元素
       */
      conditionClass: function conditionClass(element, className, bool) {
        return (bool ? CSSCore.addClass : CSSCore.removeClass)(element, className);
      },

      
      /**
       * 通过element.classList.contains方法或者element.className.indexOf，判断是否包含样式className
       * @param {DOMElement}  dom元素
       * @param {string}      判断是否的样式；只允许输入一个样式
       * @return {boolean} 
       */
      hasClass: function hasClass(element, className) {
        !!/\s/.test(className) ? process.env.NODE_ENV !== 'production' ? 
          invariant(false, 'CSS.hasClass takes only a single class name.') 
          : invariant(false) : void 0;

        if (element.classList) {
          return !!className && element.classList.contains(className);
        }
        return (' ' + element.className + ' ').indexOf(' ' + className + ' ') > -1;
      },

      /**
       * 判断element是否匹配selector选择器
       * @param {DOMNode|DOMWindow} dom元素
       * @param {string} selector   选择器
       * @return {boolean}
       */
      matchesSelector: function matchesSelector(element, selector) {
        var matchesImpl = element.matches || element.webkitMatchesSelector || element.mozMatchesSelector 
          || element.msMatchesSelector || function (s) {
          return matchesSelector_SLOW(element, s);
        };
        return matchesImpl.call(element, selector);
      }

    };

    module.exports = CSSCore;