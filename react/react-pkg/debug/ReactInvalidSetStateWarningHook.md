# ReactInvalidSetStateWarningHook::react-dom

## 概述

debug相关。

校验setState方法有否在getChildContext方法中执行，若有，则报错；通过ReactDebugTool注入组件实例化、挂载等过程中。

## 源码

    'use strict';
    
    var warning = require('fbjs/lib/warning');
    
    if (process.env.NODE_ENV !== 'production') {
      var processingChildContext = false;
    
      var warnInvalidSetState = function () {
        process.env.NODE_ENV !== 'production' ? 
          warning(!processingChildContext, 'setState(...): Cannot call setState() inside getChildContext()') : 
          void 0;
      };
    }
    
    // 校验setState方法有否在getChildContext方法中执行，若有，则报错；通过ReactDebugTool注入组件实例化、挂载等过程中
    var ReactInvalidSetStateWarningHook = {
      onBeginProcessingChildContext: function () {
        processingChildContext = true;
      },
      onEndProcessingChildContext: function () {
        processingChildContext = false;
      },
      onSetState: function () {
        warnInvalidSetState();
      }
    };
    
    module.exports = ReactInvalidSetStateWarningHook;