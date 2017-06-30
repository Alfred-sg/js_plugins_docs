# shouldUpdateReactComponent::react-dom

## 概述

react默认的shouldUpdateReactComponent生命周期方法。

shouldUpdateReactComponent(prevElement,nextElement)，返回真值，重绘时更新组件实例；否值，销毁实例后重新创建。组件的构造函数或key值不同，销毁实例后再行创建。

## 源码

    'use strict';
    
    // 判断组件重绘时是采用更新组件实例的方式(返回真值)；还是采用销毁实例后、重新创建实例的方式(返回否值)
    // 组件元素的构造函数或key值不同，销毁实例后再行创建
    function shouldUpdateReactComponent(prevElement, nextElement) {
        var prevEmpty = prevElement === null || prevElement === false;
        var nextEmpty = nextElement === null || nextElement === false;
    
        // prevElement、nextElement均为null或false
        if (prevEmpty || nextEmpty) {
            return prevEmpty === nextEmpty;
        }
    
        var prevType = typeof prevElement;
        var nextType = typeof nextElement;
    
        // prevElement、nextElement均为字符串或数值
        if (prevType === 'string' || prevType === 'number') {
            return nextType === 'string' || nextType === 'number';
        } else {
            return nextType === 'object' && prevElement.type === nextElement.type && prevElement.key === nextElement.key;
        }
    }
    
    module.exports = shouldUpdateReactComponent;
    