# PromiseMap

new PromiseMap()，以key键构建多个deferred实例，处理异步逻辑。

## 原型方法

* get(key)，获取跟key键对应的promise实例。
* resolveKey(key，value)，调用genkey键对应的promise实例的成功回调，参数为value。
* rejectKey(key，value)，调用genkey键对应的promise实例的失败回调，参数为value。

## 源码

    'use strict';

    function _classCallCheck(instance, Constructor) { if (!(instance instanceof Constructor)) { throw new TypeError("Cannot call a class as a function"); } }

    var Deferred = require('./Deferred');

    var invariant = require('./invariant');

    // 以key键构建多个deferred实例，通过resolveKey、rejectKey调用成功回调；通过get(key).then添加回调
    var PromiseMap = function () {
      function PromiseMap() {
        _classCallCheck(this, PromiseMap);

        this._deferred = {};
      }

      PromiseMap.prototype.get = function get(key) {
        return getDeferred(this._deferred, key).getPromise();
      };

      PromiseMap.prototype.resolveKey = function resolveKey(key, value) {
        var entry = getDeferred(this._deferred, key);

        // 延迟函数已执行完成，报错
        !!entry.isSettled() ? process.env.NODE_ENV !== 'production' ? 
          invariant(false, 'PromiseMap: Already settled `%s`.', key) : invariant(false) : void 0;

        entry.resolve(value);
      };

      PromiseMap.prototype.rejectKey = function rejectKey(key, reason) {
        var entry = getDeferred(this._deferred, key);
        !!entry.isSettled() ? process.env.NODE_ENV !== 'production' ? 
          invariant(false, 'PromiseMap: Already settled `%s`.', key) : invariant(false) : void 0;
        entry.reject(reason);
      };

      return PromiseMap;
    }();

    // 创建Deferred实例，并返回；或者获取已创建的Deferred实例
    function getDeferred(entries, key) {
      if (!entries.hasOwnProperty(key)) {
        entries[key] = new Deferred();
      }
      return entries[key];
    }

    module.exports = PromiseMap;
