# ReactDebugTool::react-dom

## 概述

debug相关。

在组件的生命周期方法执行过程中，校验生命周期方法运行时不与另一个生命周期方法相冲突，且组件已实例化，并缓存生命周期方法的运行时间，记录创建的节点和根节点，节点样式、节点内容、节点属性变更数据，以及校验getChildContext方法内容有否调用setState方法，有则报错。

## 源码

    'use strict';
    
    // 校验setState方法有否在getChildContext方法中执行，若有，则报错；通过ReactDebugTool注入组件实例化、挂载等过程中
    var ReactInvalidSetStateWarningHook = require('./ReactInvalidSetStateWarningHook');
    
    // 变更样式、节点、节点属性缓存操作信息；通过ReactDebugTool注入组件实例化、挂载等过程中
    var ReactHostOperationHistoryHook = require('./ReactHostOperationHistoryHook');
    
    // debug模式即本地开始时，用于获取节点的构建信息；通过ReactDebugTool注入组件实例化、挂载等过程中
    var ReactComponentTreeHook = require('react/lib/ReactComponentTreeHook');
    
    var ExecutionEnvironment = require('fbjs/lib/ExecutionEnvironment');
    var performanceNow = require('fbjs/lib/performanceNow');
    var warning = require('fbjs/lib/warning');
    
    var hooks = [];
    var didHookThrowForEvent = {};
    
    // 钩子函数首次报错时，捕获错误，并改变错误提示；之后均忽略
    function callHook(event, fn, context, arg1, arg2, arg3, arg4, arg5) {
      try {
        fn.call(context, arg1, arg2, arg3, arg4, arg5);
      } catch (e) {
        process.env.NODE_ENV !== 'production' ? 
          warning(didHookThrowForEvent[event], 'Exception thrown by hook while handling %s: %s', event, 
            e + '\n' + e.stack) : void 0;
        didHookThrowForEvent[event] = true;
      }
    }
    
    // 执行钩子函数。钩子hook以对象形式形式添加，event实际是hook的属性；由ReactDebugTool模块控制钩子的执行流程
    // 多个钩子的event若同名，调用emitEvent函数将执行所有钩子的event方法
    function emitEvent(event, arg1, arg2, arg3, arg4, arg5) {
      for (var i = 0; i < hooks.length; i++) {
        var hook = hooks[i];
        var fn = hook[event];
        if (fn) {
          callHook(event, fn, hook, arg1, arg2, arg3, arg4, arg5);
        }
      }
    }
    
    var isProfiling = false;// 状态标识，是否启用ReactHostOperationHistoryHook钩子，以监听样式、节点、节点属性变更
    var flushHistory = [];// 样式、节点、节点属性变更数据缓存
    
    // 组件生命周期方法中调用setState、replaceState、forceUpdate方法时，缓存生命周期方法及setState等方法执行的起始时间
    var lifeCycleTimerStack = [];
    // 组件生命周期方法中调用setState、replaceState、forceUpdate方法时，currentFlushNesting缓存自增1；执行结束自减1
    var currentFlushNesting = 0;
    // 缓存组件各生命周期方法的执行时间等数据
    // 组件生命周期方法中调用setState、replaceState、forceUpdate方法且监听样式、节点、节点属性变更时，才启用该缓存
    var currentFlushMeasurements = [];
    var currentFlushStartTime = 0;// 缓存组件生命周期方法内调用setState、replaceState、forceUpdate方法的起始时间
    var currentTimerDebugID = null;// 生命周期方法执行时关联的内部组件如ReactCompositeComponent实例的debugID
    var currentTimerStartTime = 0;// 生命周期方法执行的起始事件
    var currentTimerNestedFlushDuration = 0;// 缓存组件生命周期方法中调用setState、replaceState、forceUpdate方法的执行时间
    var currentTimerType = null;// 生命周期事件名如"ctor"、"componentDidMount"缓存；阻止同步执行多个生命周期方法
    
    var lifeCycleTimerHasWarned = false;// react内部生命周期执行机制错误，只警告一次
    
    // 移除ReactComponentTreeHook已卸载节点缓存、ReactHostOperationHistoryHook样式、节点、节点属性变更缓存
    function clearHistory() {
      // 移除ReactComponentTreeHook所有已卸载节点及其子节点的缓存数据
      ReactComponentTreeHook.purgeUnmountedComponents();
      // 清空ReactHostOperationHistoryHook模块中缓存的样式、节点变更、节点属性变更操作信息
      ReactHostOperationHistoryHook.clearHistory();
    }
    
    // 获取ReactElement的数据信息
    function getTreeSnapshot(registeredIDs) {
      return registeredIDs.reduce(function (tree, id) {
        var ownerID = ReactComponentTreeHook.getOwnerID(id);
        var parentID = ReactComponentTreeHook.getParentID(id);
        tree[id] = {
          displayName: ReactComponentTreeHook.getDisplayName(id),
          text: ReactComponentTreeHook.getText(id),
          updateCount: ReactComponentTreeHook.getUpdateCount(id),
          childIDs: ReactComponentTreeHook.getChildIDs(id),
          ownerID: ownerID || parentID && ReactComponentTreeHook.getOwnerID(parentID) || 0,
          parentID: parentID
        };
        return tree;
      }, {});
    }
    
    // 组件生命周期方法内调用setState、replaceState、forceUpdate方法时，重设currentFlushMeasurements
    // 并将多个setState方法的样式、节点、节点属性变更信息存入flushHistory中
    function resetMeasurements() {
      var previousStartTime = currentFlushStartTime;
      var previousMeasurements = currentFlushMeasurements;
      var previousOperations = ReactHostOperationHistoryHook.getHistory();
    
      // 组件生命周期方法内调用setState、replaceState、forceUpdate方法后执行
      if (currentFlushNesting === 0) {
        currentFlushStartTime = 0;
        currentFlushMeasurements = [];
    
        // 移除ReactComponentTreeHook已卸载节点缓存、ReactHostOperationHistoryHook样式、节点、节点属性变更缓存
        clearHistory();
        return;
      }
    
      if (previousMeasurements.length || previousOperations.length) {
        // 获取已挂载的节点缓存数据
        var registeredIDs = ReactComponentTreeHook.getRegisteredIDs();
    
        // 缓存样式、节点、节点属性变更数据
        flushHistory.push({
          duration: performanceNow() - previousStartTime,
          measurements: previousMeasurements || [],
          operations: previousOperations || [],
          treeSnapshot: getTreeSnapshot(registeredIDs)
        });
      }
    
      // 移除ReactComponentTreeHook已卸载节点缓存、ReactHostOperationHistoryHook样式、节点、节点属性变更缓存
      clearHistory();
      currentFlushStartTime = performanceNow();
      currentFlushMeasurements = [];
    }
    
    // 校验根节点或其他节点绘制过程是否有debugID
    function checkDebugID(debugID) {
      var allowRoot = arguments.length > 1 && arguments[1] !== undefined ? arguments[1] : false;
    
      if (allowRoot && debugID === 0) {
        return;
      }
      if (!debugID) {
        process.env.NODE_ENV !== 'production' ? 
          warning(false, 'ReactDebugTool: debugID may not be empty.') : void 0;
      }
    }
    
    // 组件生命周期方法执行前校验有否另一个生命周期方法正在执行中；并缓存起始执行时间
    function beginLifeCycleTimer(debugID, timerType) {
      if (currentFlushNesting === 0) {
        return;
      }
      if (currentTimerType && !lifeCycleTimerHasWarned) {
        process.env.NODE_ENV !== 'production' ? 
          warning(false, 'There is an internal error in the React performance measurement code. ' 
            + 'Did not expect %s timer to start while %s timer is still in ' 
            + 'progress for %s instance.', timerType, currentTimerType || 'no', 
            debugID === currentTimerDebugID ? 'the same' : 'another') : void 0;
        lifeCycleTimerHasWarned = true;
      }
      currentTimerStartTime = performanceNow();
      currentTimerNestedFlushDuration = 0;
      currentTimerDebugID = debugID;
      currentTimerType = timerType;
    }
    
    // 组件生命周期方法执行后校验，并将生命周期方法执行时间信息存入currentFlushMeasurements中
    function endLifeCycleTimer(debugID, timerType) {
      if (currentFlushNesting === 0) {
        return;
      }
      if (currentTimerType !== timerType && !lifeCycleTimerHasWarned) {
        process.env.NODE_ENV !== 'production' ? 
          warning(false, 'There is an internal error in the React performance measurement code. ' 
            + 'We did not expect %s timer to stop while %s timer is still in ' 
            + 'progress for %s instance. Please report this as a bug in React.', 
            timerType, currentTimerType || 'no', debugID === currentTimerDebugID ? 'the same' : 'another') : void 0;
        lifeCycleTimerHasWarned = true;
      }
      if (isProfiling) {
        currentFlushMeasurements.push({
          timerType: timerType,
          instanceID: debugID,
          duration: performanceNow() - currentTimerStartTime - currentTimerNestedFlushDuration
        });
      }
      currentTimerStartTime = 0;
      currentTimerNestedFlushDuration = 0;
      currentTimerDebugID = null;
      currentTimerType = null;
    }
    
    // 组件生命周期方法中调用setState、replaceState、forceUpdate方法时，触发执行pauseCurrentLifeCycleTimer函数
    // react实现中，通过ReactCompositeComponent将updater切面函数控制器传入组件，并包装setState、replaceState、forceUpdate方法
    //    当setState、replaceState、forceUpdate方法，将执行updater中的前置、后置钩子函数
    //    pauseCurrentLifeCycleTimer函数作为前置钩子函数的部分内容
    function pauseCurrentLifeCycleTimer() {
      var currentTimer = {
        startTime: currentTimerStartTime,
        nestedFlushStartTime: performanceNow(),
        debugID: currentTimerDebugID,
        timerType: currentTimerType
      };
      lifeCycleTimerStack.push(currentTimer);
      currentTimerStartTime = 0;
      currentTimerNestedFlushDuration = 0;
      currentTimerDebugID = null;
      currentTimerType = null;
    }
    
    // 组件生命周期方法中调用setState、replaceState、forceUpdate方法后，触发执行resumeCurrentLifeCycleTimer函数
    function resumeCurrentLifeCycleTimer() {
      var _lifeCycleTimerStack$ = lifeCycleTimerStack.pop(),
          startTime = _lifeCycleTimerStack$.startTime,
          nestedFlushStartTime = _lifeCycleTimerStack$.nestedFlushStartTime,
          debugID = _lifeCycleTimerStack$.debugID,
          timerType = _lifeCycleTimerStack$.timerType;
    
      var nestedFlushDuration = performanceNow() - nestedFlushStartTime;
      currentTimerStartTime = startTime;
      currentTimerNestedFlushDuration += nestedFlushDuration;
      currentTimerDebugID = debugID;
      currentTimerType = timerType;
    }
    
    var lastMarkTimeStamp = 0;// 缓存最近一次生命周期方法执行的时间
    var canUsePerformanceMeasure = 
      typeof performance !== 'undefined' && typeof performance.mark === 'function' && 
      typeof performance.clearMarks === 'function' && typeof performance.measure === 'function' && 
      typeof performance.clearMeasures === 'function';
    
    // debugID对应节点为ReactElement，须记录生命周期执行起始时间
    function shouldMark(debugID) {
      if (!isProfiling || !canUsePerformanceMeasure) {
        return false;
      }
      var element = ReactComponentTreeHook.getElement(debugID);
      if (element == null || typeof element !== 'object') {
        return false;
      }
      var isHostElement = typeof element.type === 'string';
      if (isHostElement) {
        return false;
      }
      return true;
    }
    
    // 缓存生命周期起始时间；performance缓存的数据可在组件的生命周期方法中通过访问performance获取
    function markBegin(debugID, markType) {
      if (!shouldMark(debugID)) {
        return;
      }
    
      var markName = debugID + '::' + markType;
      lastMarkTimeStamp = performanceNow();
      performance.mark(markName);// 以markName标记保存一个时间戳
    }
    
    // 清空生命周期起始时间缓存记录
    function markEnd(debugID, markType) {
      if (!shouldMark(debugID)) {
        return;
      }
    
      var markName = debugID + '::' + markType;
      var displayName = ReactComponentTreeHook.getDisplayName(debugID) || 'Unknown';
    
      var timeStamp = performanceNow();
      if (timeStamp - lastMarkTimeStamp > 0.1) {
        var measurementName = displayName + ' [' + markType + ']';
    
        // performance.measure(measureName,markName1,markName2)
        // 计算mark方法添加markName1、markName2的差值，并以measureName标记保存该差值
        performance.measure(measurementName, markName);
      }
    
      performance.clearMarks(markName);
      performance.clearMeasures(measurementName);
    }
    
    var ReactDebugTool = {
      // 添加钩子函数。钩子hook以对象形式形式添加，event实际是hook的属性
      addHook: function (hook) {
        hooks.push(hook);
      },
    
      // 移除钩子函数
      removeHook: function (hook) {
        for (var i = 0; i < hooks.length; i++) {
          if (hooks[i] === hook) {
            hooks.splice(i, 1);
            i--;
          }
        }
      },
    
      // 判断是否启用ReactHostOperationHistoryHook钩子，监听样式、节点、节点属性变更
      isProfiling: function () {
        return isProfiling;
      },
    
      // 添加样式、节点、节点属性变更监听钩子，ReactHostOperationHistoryHook缓存中作记录
      beginProfiling: function () {
        if (isProfiling) {
          return;
        }
    
        isProfiling = true;
        flushHistory.length = 0;
        resetMeasurements();
        ReactDebugTool.addHook(ReactHostOperationHistoryHook);
      },
    
      // 移除样式、节点、节点属性变更监听钩子
      endProfiling: function () {
        if (!isProfiling) {
          return;
        }
    
        isProfiling = false;
        resetMeasurements();
        ReactDebugTool.removeHook(ReactHostOperationHistoryHook);
      },
    
      // 获取缓存的样式、节点、节点属性变更数据
      getFlushHistory: function () {
        return flushHistory;
      },
    
      // 组件生命周期方法中调用setState、replaceState、forceUpdate方法前，更新缓存信息数据
      onBeginFlush: function () {
        currentFlushNesting++;
        resetMeasurements();
        pauseCurrentLifeCycleTimer();
    
        // 默认hook中无'onBeginFlush'方法
        emitEvent('onBeginFlush');
      },
    
      // 组件生命周期方法中调用setState、replaceState、forceUpdate方法后，更新缓存信息数据
      onEndFlush: function () {
        resetMeasurements();
        currentFlushNesting--;
        resumeCurrentLifeCycleTimer();
    
        // 默认hook中无'onEndFlush'方法
        emitEvent('onEndFlush');
      },
    
      // 组件生命周期方法执行前调用的监听函数，timerType即"ctor"实例化、"componentWillMount"、"render"渲染、
      // "componentDidMount"、"componentWillReceiveProps"、"shouldComponentUpdate"、
      // "componentWillUpdate"、"componentDidUpdate"、"componentWillUnmount"
      onBeginLifeCycleTimer: function (debugID, timerType) {
        checkDebugID(debugID);
    
        // 默认hook中无'onBeginLifeCycleTimer'方法
        emitEvent('onBeginLifeCycleTimer', debugID, timerType);
    
        // 缓存生命周期起始时间；performance缓存的数据可在组件的生命周期方法中通过访问performance获取
        markBegin(debugID, timerType);
        beginLifeCycleTimer(debugID, timerType);
      },
    
      // 组件生命周期方法执行后调用的监听函数
      onEndLifeCycleTimer: function (debugID, timerType) {
        checkDebugID(debugID);
        endLifeCycleTimer(debugID, timerType);
    
        // 清空生命周期起始时间缓存记录
        markEnd(debugID, timerType);
    
        // 默认hook中无'onEndLifeCycleTimer'方法
        emitEvent('onEndLifeCycleTimer', debugID, timerType);
      },
    
      onBeginProcessingChildContext: function () {
        // ReactInvalidSetStateWarningHook钩子中，启动校验setState方法有否在getChildContext方法中执行
        emitEvent('onBeginProcessingChildContext');
      },
      onEndProcessingChildContext: function () {
        // ReactInvalidSetStateWarningHook钩子中，关闭校验setState方法有否在getChildContext方法中执行
        emitEvent('onEndProcessingChildContext');
      },
      // ReactInvalidSetStateWarningHook钩子中，校验setState方法有否在getChildContext方法中执行，若有，则报错
      onSetState: function () {
        emitEvent('onSetState');
      },
    
      // ReactHostOperationHistoryHook钩子中，缓存样式、节点、节点属性缓存操作信息
      onHostOperation: function (operation) {
        checkDebugID(operation.instanceID);
        emitEvent('onHostOperation', operation);
      },
    
      // ReactComponentTreeHook钩子中，缓存节点、根节点信息
      onSetChildren: function (debugID, childDebugIDs) {
        checkDebugID(debugID);
        childDebugIDs.forEach(checkDebugID);
        emitEvent('onSetChildren', debugID, childDebugIDs);
      },
      onBeforeMountComponent: function (debugID, element, parentDebugID) {
        checkDebugID(debugID);
        checkDebugID(parentDebugID, true);
        emitEvent('onBeforeMountComponent', debugID, element, parentDebugID);
        markBegin(debugID, 'mount');
      },
      onMountComponent: function (debugID) {
        checkDebugID(debugID);
        markEnd(debugID, 'mount');
        emitEvent('onMountComponent', debugID);
      },
      onBeforeUpdateComponent: function (debugID, element) {
        checkDebugID(debugID);
        emitEvent('onBeforeUpdateComponent', debugID, element);
        markBegin(debugID, 'update');
      },
      onUpdateComponent: function (debugID) {
        checkDebugID(debugID);
        markEnd(debugID, 'update');
        emitEvent('onUpdateComponent', debugID);
      },
      onBeforeUnmountComponent: function (debugID) {
        checkDebugID(debugID);
        emitEvent('onBeforeUnmountComponent', debugID);
        markBegin(debugID, 'unmount');
      },
      onUnmountComponent: function (debugID) {
        checkDebugID(debugID);
        markEnd(debugID, 'unmount');
        emitEvent('onUnmountComponent', debugID);
      },
    
      onTestEvent: function () {
        // 默认hook中无'onTestEvent'方法
        emitEvent('onTestEvent');
      }
    };
    
    ReactDebugTool.addDevtool = ReactDebugTool.addHook;
    ReactDebugTool.removeDevtool = ReactDebugTool.removeHook;
    
    ReactDebugTool.addHook(ReactInvalidSetStateWarningHook);
    ReactDebugTool.addHook(ReactComponentTreeHook);
    var url = ExecutionEnvironment.canUseDOM && window.location.href || '';
    if (/[?&]react_perf\b/.test(url)) {
      ReactDebugTool.beginProfiling();
    }
    
    module.exports = ReactDebugTool;
