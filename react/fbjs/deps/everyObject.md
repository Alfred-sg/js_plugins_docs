# everyObject

everyObject(object,callback,context)，遍历object对象的属性，以context作为上下文执行callback函数；当callback返回否值时，遍历终止。

    'use strict';

    var hasOwnProperty = Object.prototype.hasOwnProperty;

    /**
     * 遍历object对象的属性，以context作为上下文执行callback函数；当callback返回否值时，遍历终止
     * @param  {object}   object   待遍历的对象
     * @param  {Function} callback 遍历执行函数；返回否值终止遍历
     * @param  {object}   context  上下文对象
     * @return {boolean}           遍历所有属性返回true，否则返回false
     */
    function everyObject(object, callback, context) {
      for (var name in object) {
        if (hasOwnProperty.call(object, name)) {
          if (!callback.call(context, object[name], name, object)) {
            return false;
          }
        }
      }
      return true;
    }

    module.exports = everyObject;