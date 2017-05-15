# keyMirror

keyMirror(obj)，等同Object.keys(obj)获取对象的属性，却以对象形式返回对象的属性到其自身的映射。

    'use strict';

    var invariant = require('./invariant');

    // 等同Object.keys(obj)，数组形式返回对象的属性
    var keyMirror = function keyMirror(obj) {
      var ret = {};
      var key;
      !(obj instanceof Object && !Array.isArray(obj)) ? process.env.NODE_ENV !== 'production' ? 
        invariant(false, 'keyMirror(...): Argument must be an object.') : invariant(false) : void 0;
      for (key in obj) {
        if (!obj.hasOwnProperty(key)) {
          continue;
        }
        ret[key] = key;
      }
      return ret;
    };

    module.exports = keyMirror;