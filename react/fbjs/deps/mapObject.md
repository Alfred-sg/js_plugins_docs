# mapObject

mapObject(object,callback,context)，遍历object对象的属性，以context作为上下文执行callback函数。不同于forEachObject函数的无返回值，mapObject以对象形式输出回调callback执行结果。

    'use strict';

    var hasOwnProperty = Object.prototype.hasOwnProperty;

    // 遍历object对象的属性，以context作为上下文执行callback函数
    // 不同于forEachObject函数的无返回值，mapObject以对象形式输出回调callback执行结果
    function mapObject(object, callback, context) {
      if (!object) {
        return null;
      }
      var result = {};
      for (var name in object) {
        if (hasOwnProperty.call(object, name)) {
          result[name] = callback.call(context, object[name], name, object);
        }
      }
      return result;
    }

    module.exports = mapObject;