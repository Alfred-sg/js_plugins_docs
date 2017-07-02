# canDefineProperty::react-dom

校验平台是否能使用Object.defineProperty方法。

    'use strict';
    
    var canDefineProperty = false;
    if (process.env.NODE_ENV !== 'production') {
      try {
        Object.defineProperty({}, 'x', { get: function () {} });
        canDefineProperty = true;
      } catch (x) {
      }
    }
    
    module.exports = canDefineProperty;