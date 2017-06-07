# performanceNow

performanceNow()，获取页面开始加载到js代码执行时的时间戳，可用于计算代码执行的耗时。

    'use strict';
    
    var performance = require('./performance');
    
    var performanceNow;
    
    // 获取页面开始加载到js代码执行时的时间戳，可用于计算代码执行的耗时
    if (performance.now) {
      performanceNow = function performanceNow() {
        return performance.now();
      };
    } else {
      performanceNow = function performanceNow() {
        return Date.now();
      };
    }
    
    module.exports = performanceNow;