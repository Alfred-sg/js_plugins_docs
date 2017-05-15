# someObject

someObject(object,callback,context)，遍历object对象的属性，以context作为上下文执行callback函数；雷同[].some(fn)，某属性在调用callback返回真值，则最终输出true；否则为false。

    'use strict';

    var hasOwnProperty = Object.prototype.hasOwnProperty;

    // 雷同[].some(fn)，某属性在调用callback返回真值，则最终输出true；否则为false
    function someObject(object, callback, context) {
      for (var name in object) {
        if (hasOwnProperty.call(object, name)) {
          if (callback.call(context, object[name], name, object)) {
            return true;
          }
        }
      }
      return false;
    }

    module.exports = someObject;