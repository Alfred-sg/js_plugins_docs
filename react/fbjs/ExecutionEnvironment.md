# ExecutionEnvironment

浏览器环境能力侦测。

    'use strict';

    var canUseDOM = !!(typeof window !== 'undefined' && window.document && window.document.createElement);

    // 浏览器环境能力侦测
    var ExecutionEnvironment = {
      // 通过document.createElement判断createElement判断能够使用浏览器的dom操作接口
      canUseDOM: canUseDOM,
      // 通过全局对象Worker判断是否webworker，浏览器启动的js线程
      canUseWorkers: typeof Worker !== 'undefined',
      // 通过window.addEventListener或window.attachEvent判断是否使用浏览器的事件接口
      canUseEventListeners: canUseDOM && !!(window.addEventListener || window.attachEvent),
      // 通过window.screen判断是否可以获取屏幕信息
      canUseViewport: canUseDOM && !!window.screen,
      // 是否在webworker线程中
      isInWorker: !canUseDOM // For now, this is true - might change in the future.

    };

    module.exports = ExecutionEnvironment;