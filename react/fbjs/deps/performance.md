# performance

通过window.performance作性能检测用。

## 参考

[前端性能监控方案window.performance 调研(转)](http://www.cnblogs.com/sunshq/p/5312231.html)
[初探 performance - 监控网页与程序性能](http://www.07net01.com/2015/09/920822.html)

## 源码

    'use strict';
    
    var ExecutionEnvironment = require('./ExecutionEnvironment');
    
    var performance;
    
    if (ExecutionEnvironment.canUseDOM) {
      performance = window.performance || window.msPerformance || window.webkitPerformance;
    }
    
    // window.performance用于分析网页访问性能
    // window.performance={
    //      timeing:{
    //          // 浏览器渲染页面过程，依次执行
    //      
    //          // 浏览器开始unload前一个页面文档(由百度跳转到谷歌时的百度,当前文档为空，取fetchStart)的起始时间
    //          startTime|navigationStart, 
    //
    //          // 重定向起始、结束时间
    //          redirectStart,
    //          redirectEnd,
    //          
    //          // 前一个文档和请求的文档在同一个域中，unloadEventStart为unload前一个文档的开始时间，否则为0
    //          unloadEventStart,
    //          unloadEventEnd,
    //          
    //          // 浏览器发起任何请求之前的时间值；在fetchStart和domainLookupStart之间，浏览器会检查当前文档的缓存
    //          fetchStart,
    //          
    //          // DNS查询的起始、结束时间；取缓存时不用DNS查询，两者值取fetchStart
    //          domainLookupStart,
    //          domainLookupEnd,
    //          
    //          // 建立TCP连接的起始、结束时间；若无TCP连接(如使用webscoket长连接)，两者值取domainLookupEnd
    //          connectStart,
    //          [secureConnectionStart],// HTTPS协议，安全连接握手之前的时刻，否则返回0
    //          connectEnd,
    //          
    //          // 浏览器发起请求的时间，请求的方式可以是请求服务器、缓存、本地资源等
    //          requestStart
    //          
    //          // 浏览器收到服务器端、本地资源起始字节、结束字节的时间
    //          responseStart,
    //          responseEnd,
    //          
    //          // 浏览器开始解析html文档的时间，即document的readyState属性改变为loading状态时
    //          domLoading,
    //          
    //          // 解析html文档的状态为interactive的时间，早于DOMReady触发
    //          domInteractive,
    //          
    //          // DOMContentLoaded事件起始、结束时间
    //          // 结束时意味dom树已渲染完成，用户可对页面进行操作，调用document.ready事件
    //          domContentLoadedEventStart,
    //          domContentLoadedEventEnd,
    //          
    //          // html文档完全解析完毕的时间
    //          domComplete,
    //          
    //          // onload事件触发和结束的时间；结束时间即页面资源加载完成时机，用户可调用window.onload
    //          loadEventStart,
    //          loadEventEnd,
    //      },
    //      
    //      memory:{
    //          // 浏览器内存占用情况
    //          
    //          // 当前使用的js堆栈内存大小
    //          usedJSHeapSize,
    //          
    //          // 可使用的内存总大小；usedJSHeapSize若大于totalJSHeapSize，内存泄漏
    //          totalJSHeapSize,
    //          
    //          // 内存大小限制
    //          jsHeapSizeLimit
    //      },
    //      
    //      navigation:{
    //          // 页面重定向次数
    //          redirectCount,
    //          
    //          // 0即TYPE_NAVIGATENEXT，正常进入的页面；1即TYPE_RELOAD，通过点击刷新按钮或location.reload()访问
    //          // 2即TYPE_BACK_FORWARD通过前进后退按钮访问；255即TYPE_UNDEFINED，非上述方式访问
    //          type
    //      },
    //      now:()=>{},// fetchStart到现在的微秒数
    //      mark:(markName)=>{},// 以markName标记保存一个时间戳
    //      measure:(measureName,markName1,markName2),// 计算mark方法添加markName1、markName2的差值，并以measureName标记保存该差值；markName2置空，与当前时间作比较
    //      getEntriesByType:("mark|measure")=>{},// 获取所有mark方法或measure方法缓存的时间戳
    //      getEntriesByName:([markName|measureName])=>{},// 获取单个mark方法或measure方法缓存的时间戳
    //      clearMarks:([markName])=>{},// 清空特定或所有mark方法缓存的时间戳
    //      clearMeasures:([measureName])=>{},// 清空特定或所有measure方法缓存的时间戳
    // }
    
    // 计算性能指标
    // DNS查询耗时 = domainLookupEnd - domainLookupStart
    // TCP链接耗时 = connectEnd - connectStart
    // request请求耗时 = responseEnd - responseStart
    // 解析dom树耗时 = domComplete - domInteractive
    // 白屏时间 = domloadng - fetchStart
    // domready时间 = domContentLoadedEventEnd - fetchStart
    // onload时间 = loadEventEnd - fetchStart
    module.exports = performance || {};