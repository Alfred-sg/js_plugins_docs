# cx

cx({className1:true,className2:false})，将值为真的属性拼接为样式输出。
cx(className1,...calssNameN)，将多个字符串拼接为样式后输出。

    'use strict';

    /**
     * 对象(取值为真的属性拼接为样式)或多参数形式设置样式，样式中的"/"将自动转换为"-"；同classnames模块
     * @param  {object|className} classNames 单参数对象形式，取值为真的属性拼接为样式；多参数字符串形式，拼接为样式
     * @return {className}                   拼接后的样式
     */ 
    function cx(classNames) {
      if (typeof classNames == 'object') {
        return Object.keys(classNames).filter(function (className) {
          return classNames[className];
        }).map(replace).join(' ');
      }
      return Array.prototype.map.call(arguments, replace).join(' ');
    }

    function replace(str) {
      return str.replace(/\//g, '-');
    }

    module.exports = cx;