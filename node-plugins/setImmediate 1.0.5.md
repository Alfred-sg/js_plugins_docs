# setImmediate 1.0.5

## 概述

setImmediate(callback,...args)，等待强业务逻辑函数执行完成后，运行callback回调。

## 源码

    (function (global, undefined) {
        "use strict";
    
        if (global.setImmediate) {
            return;
        }
    
        var nextHandle = 1; // Spec says greater than zero
        var tasksByHandle = {};
        var currentlyRunningATask = false;
        var doc = global.document;
        var registerImmediate;
    
        // 将回调及其参数以{callback,args}形式存入tasksByHandle
        function setImmediate(callback) {
          if (typeof callback !== "function") {
            callback = new Function("" + callback);
          }
          var args = new Array(arguments.length - 1);
          for (var i = 0; i < args.length; i++) {
              args[i] = arguments[i + 1];
          }
          var task = { callback: callback, args: args };
          tasksByHandle[nextHandle] = task;
          registerImmediate(nextHandle);
          return nextHandle++;
        }
    
        // 删除特定回调及参数
        function clearImmediate(handle) {
            delete tasksByHandle[handle];
        }
    
        // 执行回调
        function run(task) {
            var callback = task.callback;
            var args = task.args;
            switch (args.length) {
            case 0:
                callback();
                break;
            case 1:
                callback(args[0]);
                break;
            case 2:
                callback(args[0], args[1]);
                break;
            case 3:
                callback(args[0], args[1], args[2]);
                break;
            default:
                callback.apply(undefined, args);
                break;
            }
        }
    
        // 有回调在执行过程中，等待其执行完成后调用回调；若无，直接执行回调
        function runIfPresent(handle) {
            // 有回调函数在执行过程中，通过setTimeout自调用runIfPresent函数，等待执行的回调完成后，执行下一个回调
            if (currentlyRunningATask) {
                setTimeout(runIfPresent, 0, handle);
            } else {
                var task = tasksByHandle[handle];
                if (task) {
                    currentlyRunningATask = true;
                    try {
                        run(task);
                    } finally {
                        clearImmediate(handle);
                        currentlyRunningATask = false;
                    }
                }
            }
        }
    
        // node端调用process.nextTick执行回调
        function installNextTickImplementation() {
            registerImmediate = function(handle) {
                process.nextTick(function () { runIfPresent(handle); });
            };
        }
    
        function canUsePostMessage() {
            if (global.postMessage && !global.importScripts) {
                var postMessageIsAsynchronous = true;
                var oldOnMessage = global.onmessage;
                global.onmessage = function() {
                    postMessageIsAsynchronous = false;
                };
                global.postMessage("", "*");
                global.onmessage = oldOnMessage;
                return postMessageIsAsynchronous;
            }
        }
    
        function installPostMessageImplementation() {
            var messagePrefix = "setImmediate$" + Math.random() + "$";
            var onGlobalMessage = function(event) {
                if (event.source === global &&
                    typeof event.data === "string" &&
                    event.data.indexOf(messagePrefix) === 0) {
                    runIfPresent(+event.data.slice(messagePrefix.length));
                }
            };
    
            if (global.addEventListener) {
                global.addEventListener("message", onGlobalMessage, false);
            } else {
                global.attachEvent("onmessage", onGlobalMessage);
            }
    
            registerImmediate = function(handle) {
                global.postMessage(messagePrefix + handle, "*");
            };
        }
    
        function installMessageChannelImplementation() {
            var channel = new MessageChannel();
            channel.port1.onmessage = function(event) {
                var handle = event.data;
                runIfPresent(handle);
            };
    
            registerImmediate = function(handle) {
                channel.port2.postMessage(handle);
            };
        }
    
        function installReadyStateChangeImplementation() {
            var html = doc.documentElement;
            registerImmediate = function(handle) {
                var script = doc.createElement("script");
                script.onreadystatechange = function () {
                    runIfPresent(handle);
                    script.onreadystatechange = null;
                    html.removeChild(script);
                    script = null;
                };
                html.appendChild(script);
            };
        }
    
        function installSetTimeoutImplementation() {
            registerImmediate = function(handle) {
                setTimeout(runIfPresent, 0, handle);
            };
        }
    
        var attachTo = Object.getPrototypeOf && Object.getPrototypeOf(global);
        attachTo = attachTo && attachTo.setTimeout ? attachTo : global;
    
        // node端
        if ({}.toString.call(global.process) === "[object process]") {
            // For Node.js before 0.9
            installNextTickImplementation();
    
        // non-IE10等现代浏览器
        } else if (canUsePostMessage()) {
            installPostMessageImplementation();
    
        // webworkers
        } else if (global.MessageChannel) {
            installMessageChannelImplementation();
    
        // IE 6–8
        } else if (doc && "onreadystatechange" in doc.createElement("script")) {
            installReadyStateChangeImplementation();
    
        // 老浏览器
        } else {
            installSetTimeoutImplementation();
        }
    
        attachTo.setImmediate = setImmediate;
        attachTo.clearImmediate = clearImmediate;
    }(typeof self === "undefined" ? typeof global === "undefined" ? this : global : self));
