# getNextDebugID::react-dom

## 概述

debug相关。

自增1生成debugId。

## 源码

    'use strict';
    
    var nextDebugID = 1;
    
    function getNextDebugID() {
      return nextDebugID++;
    }
    
    // 自增1生成debugId
    module.exports = getNextDebugID;