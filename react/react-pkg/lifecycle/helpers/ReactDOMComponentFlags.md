# ReactDOMComponentFlags::react-dom

## 概述

用于判断ReactDomComponent及ReactDomEmptyComponent实例同其对应节点的铰链状态。ReactDOMComponentTree模块中使用，未铰链，则执行铰链操作；已铰链，则跳过。

## 源码

    'use strict';
    
    var ReactDOMComponentFlags = {
      hasCachedChildNodes: 1 << 0// 即1
    };
    
    // ReactDomComponent及ReactDomEmptyComponent实例挂载前后用于更新同其节点铰链标识inst._flags属性
    // 未铰链为0，已铰链为1，同ReactDOMComponentFlags.hasCachedChildNodes按位与即可判断铰链状态
    module.exports = ReactDOMComponentFlags;