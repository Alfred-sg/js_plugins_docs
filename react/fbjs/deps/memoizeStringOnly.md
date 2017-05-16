# memoizeStringOnly

memoizeStringOnly(callback)，返回(key)=>{}，调用callback回调设置cache[key]闭包缓存、或者获取cache[key]闭包缓存。

    'use strict';

    /**
     * 用于设置或获取闭包缓存，callback回调通过key键设置缓存的数据；返回以key键为传参获取或设置相应缓存的函数
     * @param  {Function} callback 获取存入cache[key]中的数据
     * @return {Function}          (key)=>{}获取或设置cache[key]缓存
     */
    function memoizeStringOnly(callback) {
      var cache = {};
      return function (string) {
        if (!cache.hasOwnProperty(string)) {
          cache[string] = callback.call(this, string);
        }
        return cache[string];
      };
    }

    module.exports = memoizeStringOnly;