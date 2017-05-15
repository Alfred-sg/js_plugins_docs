# forEachObject

forEachObject(object,callback,context)，遍历object对象的属性，以context作为上下文执行callback函数。

    'use strict';

    var hasOwnProperty = Object.prototype.hasOwnProperty;

    /**
    * 遍历object对象的属性，以context作为上下文执行callback函数
    * @param  {object}   object   待遍历的对象
    * @param  {Function} callback 遍历执行函数
    * @param  {object}   context  上下文对象
    */
    function forEachObject(object, callback, context) {
      for (var name in object) {
        if (hasOwnProperty.call(object, name)) {
          callback.call(context, object[name], name, object);
        }
      }
    }

    module.exports = forEachObject;