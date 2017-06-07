# resolveImmediate

resolveImmediate(callback)，通过promise机制执行callback回调函数。

    "use strict";
    
    var Promise = require("./Promise");
    
    // 通过伪延迟函数构建promise的工厂函数，传给成功回调的参数为undefined
    var resolvedPromise = Promise.resolve();
    
    // 通过promise机制执行callback回调函数
    function resolveImmediate(callback) {
      resolvedPromise.then(callback)["catch"](throwNext);
    }
    
    function throwNext(error) {
      setTimeout(function () {
        throw error;
      }, 0);
    }
    
    module.exports = resolveImmediate;