# nativeRequestAnimationFrame

nativeRequestAnimationFrame(cb)，cb中操纵dom节点实现动画，借助window.requestAnimationFrame实现。

    "use strict";
    
    var nativeRequestAnimationFrame = global.requestAnimationFrame || global.webkitRequestAnimationFrame || global.mozRequestAnimationFrame || global.oRequestAnimationFrame || global.msRequestAnimationFrame;
    
    module.exports = nativeRequestAnimationFrame;