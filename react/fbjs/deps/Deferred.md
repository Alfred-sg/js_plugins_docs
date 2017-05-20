# Deferred

new Deferred()，以jquery方式处理异步逻辑。

## 原型方法

* getPromise()获取promise实例
* resolve(arg)使promise状态变更为resolved，并为成功回调传入参数arg。
* reject(arg)使promise状态变更为rejected，并为失败回调传入参数arg。
* catch(onRejected)失败状态执行回调onRejected。
* then(onFulfilled,onRejected)设定成功、失败回调。
* done(onFulfilled,onRejected)同then方法，设定成功、失败回调；遇错时报错。
* isSettled()判断延迟函数是否执行完成。

## 源码

    "use strict";

    var Promise = require("./Promise");

    function _classCallCheck(instance, Constructor) { if (!(instance instanceof Constructor)) { throw new TypeError("Cannot call a class as a function"); } }

    // jquery方式处理异步逻辑
    // 
    // let def = new Deferred();
    // 
    // let async = ()=>{
    //   setTimeout(()=>{
    //     def.resolve();
    //   },300);
    //   
    //   return def;
    // };
    // 
    // async().then(onFulfilled,onRejected);
    var Deferred = function () {
    function Deferred() {
      var _this = this;

      _classCallCheck(this, Deferred);

      this._settled = false;
      this._promise = new Promise(function (resolve, reject) {
        _this._resolve = resolve;
        _this._reject = reject;
      });
    }

    Deferred.prototype.getPromise = function getPromise() {
      return this._promise;
    };

    Deferred.prototype.resolve = function resolve(value) {
      this._settled = true;
      this._resolve(value);
    };

    Deferred.prototype.reject = function reject(reason) {
      this._settled = true;
      this._reject(reason);
    };

    Deferred.prototype["catch"] = function _catch() {
      return Promise.prototype["catch"].apply(this._promise, arguments);
    };

    Deferred.prototype.then = function then() {
      return Promise.prototype.then.apply(this._promise, arguments);
    };

    Deferred.prototype.done = function done() {
      var promise = arguments.length ? this._promise.then.apply(this._promise, arguments) : this._promise;
      promise.then(undefined, function (err) {
        setTimeout(function () {
          throw err;
        }, 0);
      });
    };

    // 判断延迟函数是否执行完成
    Deferred.prototype.isSettled = function isSettled() {
      return this._settled;
    };

    return Deferred;
    }();

    module.exports = Deferred;