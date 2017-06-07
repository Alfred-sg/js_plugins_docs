# setImmediate

setImmediate(callback,...args)，等待强业务逻辑函数执行完成后，运行callback回调。

    'use strict';
    
    require('setimmediate');
    module.exports = global.setImmediate;