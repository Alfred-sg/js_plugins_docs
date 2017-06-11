# requestAnimationFrame

requestAnimationFrame(cb)，当浏览器不支持window.requestAnimationFrame，通过setTimeout实现延迟执行回调cb，以实现动画。

    'use strict';
    
    var emptyFunction = require('./emptyFunction');
    var nativeRequestAnimationFrame = require('./nativeRequestAnimationFrame');
    
    var lastTime = 0;
    
    var requestAnimationFrame = nativeRequestAnimationFrame || function (callback) {
      var currTime = Date.now();
      var timeDelay = Math.max(0, 16 - (currTime - lastTime));
      lastTime = currTime + timeDelay;
      return global.setTimeout(function () {
        callback(Date.now());
      }, timeDelay);
    };
    
    // Works around a rare bug in Safari 6 where the first request is never invoked.
    requestAnimationFrame(emptyFunction);
    
    module.exports = requestAnimationFrame;