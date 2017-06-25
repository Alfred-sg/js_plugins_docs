# ReactInstrumentation::react-dom

## 概述

debug相关。

确保ReactDebugTool调试函数库只在本地开发环境中启用。

## 源码

    'use strict';
    
    var debugTool = null;
    
    if (process.env.NODE_ENV !== 'production') {
      var ReactDebugTool = require('./ReactDebugTool');
      debugTool = ReactDebugTool;
    }
    
    module.exports = { debugTool: debugTool };