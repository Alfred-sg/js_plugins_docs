# keyMirrorRecursive

keyMirrorRecursive(obj,prefix)，加前缀prefix获取对象属性到属性的映射；若为深度嵌套形式，则映射为属性到值对象映射属性对象。

    'use strict';

    var invariant = require('./invariant');

    // 等同keyMirror函数，获取obj对象属性key加前缀prefix到属性key的映射；若属性key的值为对象，则其值赋为嵌套对象属性的映射
    function keyMirrorRecursive(obj, prefix) {
      return keyMirrorRecursiveInternal(obj, prefix);
    }

    function keyMirrorRecursiveInternal(obj, prefix) {
      var ret = {};
      var key;

      !isObject(obj) ? process.env.NODE_ENV !== 'production' ? 
        invariant(false, 'keyMirrorRecursive(...): Argument must be an object.') : 
        invariant(false) : void 0;

      for (key in obj) {
        if (!obj.hasOwnProperty(key)) {
          continue;
        }

        var val = obj[key];

        var newPrefix = prefix ? prefix + '.' + key : key;

        if (isObject(val)) {
          val = keyMirrorRecursiveInternal(val, newPrefix);
        } else {
          val = newPrefix;
        }

        ret[key] = val;
      }
      return ret;
    }

    function isObject(obj){
      return obj instanceof Object && !Array.isArray(obj);
    }

    module.exports = keyMirrorRecursive;