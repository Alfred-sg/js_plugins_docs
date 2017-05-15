# filterObject

filterObject(object,callback,context)，遍历object对象的属性，以context作为上下文执行callback函数；雷同[].filter(fn)，callback返回真值，则最终输出对象中保留对象obj的属性。

    'use strict';

    var hasOwnProperty = Object.prototype.hasOwnProperty;

    // 雷同[].filter(fn)，callback返回真值，则最终输出对象中保留对象obj的属性
    function filterObject(object, callback, context) {
      if (!object) {
        return null;
      }
      var result = {};
      for (var name in object) {
        if (hasOwnProperty.call(object, name) && callback.call(context, object[name], name, object)) {
          result[name] = object[name];
        }
      }
      return result;
    }

    module.exports = filterObject;