# derivation.md

## 概述

derivations对应mobx的概念为衍生，状态改变后不会引起响应式动作；包含computedValue计算数据、reaction副作用。shouldCompute(derivation)判断观察的derivation.observing=[observable]数据更新时，执行reaction副作用。

## apis

### 接口

* IDerivation，derivation接口。含observing属性为更新后的observable实例数组；newObserving属性为待添加的observable实例数组；dependenciesState状态，值为IDerivationState中各枚举值；runId属性为derivation执行时的uuid；unboundDepsCount属性为未绑定的依赖个数，core/observable.js模块reportObserved函数执行时变更，更新derivation.observing时使用该属性；__mapid属性将作为observable.observersIndexes的属性名；onBecomeStale方法，core/derivation.js模块bindDependencies函数执行更新derivation.observing=[observable]时，若[observable]数组某项的dependenciesState属性为-1时调用该方法。

### 常量与函数

* IDerivationState，枚举类型，观察数据和计算数据变更状态。IDerivationState.NOT_TRACKING=-1，初始值，数据需要更新；IDerivationState.UP_TO_DATE=0，数据已更新；IDerivationState.POSSIBLY_STALE=1，条件更新，含计算数据须更新，观察数据无需更新；IDerivationState.STALE=2，数据已过期，需要更新。
* new CaughtException(cause)，错误对象构造函数。
* isCaughtException(e)，判断e是否CaughtException实例。
* shouldCompute(derivation)，判断derivation.observing中观察数据或计算数据是否需要重新计算。若derivation为IDerivationState.UP_TO_DATE时，无需计算；若derivation为IDerivationState.NOT_TRACKING、IDerivationState.STALE时，需计算；若derivation为IDerivationState.POSSIBLY_STALE时，derivation.observing中有计算数据需重新计算，其他无需计算。
* isComputingDerivation()，通过globalState.trackingDerivation，判断是否有执行中的derivation。
* checkIfStateModificationsAreAllowed(atom)，atom.observers有值且globalState.computationDepth大于0或globalState.allowStateChanges为真值时，报错。
* trackDerivedFunction(derivation,f,context)，以上下文context执行fn函数，执行过程中globalState.trackingDerivation设为derivation。执行前后缓存globalState.trackingDerivation前一个值，并在f执行结束后将globalState.trackingDerivation回设为该值，并调用bindDependencies函数更新derivation.observing=[observable]。
* bindDependencies(derivation)，更新derivation.observing=[observable]，并更新derivation.observing=[IObservable]数组项与derivation的关联性。若derivation.observing某数组项的observable的dependenciesState为-1时，derivation.dependenciesState置为-1，并执行derivation.onBecomeStale方法。
* clearObserving(derivation)，重置derivation，清空derivation.observing数组，derivation.observing=[IObservable]数组项observable实例移除derivation，并将derivation.dependenciesState置为-1。
* untracked(action)，执行action，执行前后缓存globalState.trackingDerivation前一个值，并在action执行结束后回设该值。
* untrackedStart()，取出globalState.trackingDerivation，且将globalState.trackingDerivation置为null。
* untrackedEnd(prev)，将globalState.trackingDerivation设为prev，prev通常是untrackedStart函数取出的globalState.trackingDerivation，即前一个值。
* changeDependenciesStateTo0(derivation)，将derivation.dependenciesState置为0，derivation.observing=[IObservable]数组项的lowestObserverState置为0。

## 源码

    import {
        // IObservable接口，须实现name、observing、diffValue、lastAccessedBy、lowestObserverState
        // 及isPendingUnobservation、observers、observersIndexes属性、onBecomeUnobserved方法 
        IObservable,
        IDepTreeNode,// IDepTreeNode接口，须实现name、observing属性，其中observing属性为实现IObservable接口的对象数组 
        addObserver,// addObserver(observable,node)，向IObservable实例observable的属性observers添加IDerivation实例 
        removeObserver// removeObserver(observable,node)，从observable的属性observersIndexes移除node.__mapid映射
                // 若observable.observers均移除，将observable.isPendingUnobservation置为true，并向globalState.pendingUnobservations添加observable
    } from "./observable";
    import {
        IAtom// IAtom接口，继承IObservable接口，须实现name、observing、diffValue、lastAccessedBy、lowestObserverState
                // 及isPendingUnobservation、observers、observersIndexes属性、onBecomeUnobserved方法
    } from "./atom";
    import {
        globalState// 全局缓存globalState
    } from "./globalstate";
    import {
        fail// fail(message,thing)，报错
    } from "../utils/utils";
    import {
        isComputedValue
    } from "./computedvalue";
    import {
        getMessage// getMessage(id)，获取提示信息
    } from "../utils/messages";
    
    // IDerivationState观察数据和计算数据变更状态
    export enum IDerivationState {
        NOT_TRACKING = -1,// 初始值，数据需要更新
        UP_TO_DATE = 0,// 数据已更新
        POSSIBLY_STALE = 1,// 条件更新数据
        STALE = 2// 数据已过期，需要更新
    }
    
    export interface IDerivation extends IDepTreeNode {
        // 当前或待移除的[observable]
        observing: IObservable[];
    
        // 待添加的[observable]
        newObserving: null | IObservable[];
    
        // 更新状态
        dependenciesState: IDerivationState;
        
        // 作为执行中derivation的uuid
        runId: number;
    
        // core/observable.js模块下reportObserved函数执行时改变unboundDepsCount属性的值
        // 未绑定的依赖个数
        unboundDepsCount: number;
    
        // 作为observable.observersIndexes的属性名
        __mapid: string;
    
        // bindDependencies函数执行更新derivation.observing=[observable]某数组项下dependenciesState属性为-1时调用
        onBecomeStale();
    }
    
    // CaughtException类，错误对象构造函数
    export class CaughtException {
        constructor(public cause: any) { }
    }
    
    // isCaughtException(e)，判断e是否CaughtException实例
    export function isCaughtException(e): e is CaughtException {
        return e instanceof CaughtException;
    }
    
    // shouldCompute(derivation)
    // 判断derivation.observing中观察数据或计算数据是否需要重新计算
    // derivation为IDerivationState.UP_TO_DATE时，无需计算
    // derivation为IDerivationState.NOT_TRACKING、IDerivationState.STALE时，需计算
    // derivation为IDerivationState.POSSIBLY_STALE时，derivation.observing中有计算数据需重新计算，其他无需计算
    export function shouldCompute(derivation: IDerivation): boolean {
        switch (derivation.dependenciesState) {
            case IDerivationState.UP_TO_DATE: 
                return false;
    
            case IDerivationState.NOT_TRACKING: 
            case IDerivationState.STALE: 
              return true;
    
            case IDerivationState.POSSIBLY_STALE: {
                // 取出globalState.trackingDerivation
                const prevUntracked = untrackedStart();
                const obs = derivation.observing, l = obs.length;
                for (let i = 0; i < l; i++) {
                    const obj = obs[i];
                    if (isComputedValue(obj)) {
                        try {
                            obj.get();// 可能改变derivation.dependenciesState的值
                        } catch (e) {
                            // 将globalState.trackingDerivation设为prevUntracked，即取出的globalState.trackingDerivation
                            untrackedEnd(prevUntracked);
                            return true;
                        }
    
                        if ((derivation as any).dependenciesState === IDerivationState.STALE) {
                            untrackedEnd(prevUntracked);
                            return true;
                        }
                    }
                }
                changeDependenciesStateTo0(derivation);
                untrackedEnd(prevUntracked);
                return false;
            }
            
        }
    }
    
    // isComputingDerivation()
    // 通过globalState.trackingDerivation，判断是否有执行中的derivation
    export function isComputingDerivation() {
        return globalState.trackingDerivation !== null;
    }
    
    // checkIfStateModificationsAreAllowed(atom)
    // atom.observers有值且globalState.computationDepth大于0或globalState.allowStateChanges为真值时，报错
    // ???
    export function checkIfStateModificationsAreAllowed(atom: IAtom) {
        const hasObservers = atom.observers.length > 0;
    
        // core/computedvalue.js模块中，ComputedValue实例重新计算新值时globalState.computationDepth > 0
        if (globalState.computationDepth > 0 && hasObservers)
            fail(getMessage("m031") + atom.name);
        
        if (!globalState.allowStateChanges && hasObservers)
            fail(getMessage(globalState.strictMode ? "m030a" : "m030b") + atom.name);
    }
    
    // trackDerivedFunction(derivation,f,context)
    // 以上下文context执行fn函数，执行过程中globalState.trackingDerivation设为derivation
    // 执行前后缓存globalState.trackingDerivation前一个值，并在f执行结束后回设该值
    // 并调用bindDependencies函数更新derivation.observing=[observable]
    export function trackDerivedFunction<T>(derivation: IDerivation, f: () => T, context) {
        changeDependenciesStateTo0(derivation);
        derivation.newObserving = new Array(derivation.observing.length + 100);
        derivation.unboundDepsCount = 0;
        derivation.runId = ++globalState.runId;
        const prevTracking = globalState.trackingDerivation;
        globalState.trackingDerivation = derivation;
        let result;
        try {
            result = f.call(context);
        } catch (e) {
            result = new CaughtException(e);
        }
        globalState.trackingDerivation = prevTracking;
        bindDependencies(derivation);
        return result;
    }
    
    // bindDependencies(derivation)
    // 更新derivation.observing=[observable]，并更新derivation.observing=[IObservable]数组项与derivation的关联性
    // 若derivation.observing某数组项的observable的dependenciesState为-1时
    //      derivation.dependenciesState置为-1，并执行derivation.onBecomeStale方法
    function bindDependencies(derivation: IDerivation) {
        const prevObserving = derivation.observing;// 待移除的[observable]
        const observing = derivation.observing = derivation.newObserving!;// 待添加的[observable]
        let lowestNewObservingDerivationState = IDerivationState.UP_TO_DATE;
    
        derivation.newObserving = null;
    
        let i0 = 0, l = derivation.unboundDepsCount;
        for (let i = 0; i < l; i++) {
            const dep = observing[i];
            if (dep.diffValue === 0) {
                dep.diffValue = 1;
                if (i0 !== i) observing[i0] = dep;
                i0++;
            }
    
            if ((dep as any as IDerivation).dependenciesState > lowestNewObservingDerivationState) {
                lowestNewObservingDerivationState = (dep as any as IDerivation).dependenciesState;
            }
        }
        observing.length = i0;
    
        l = prevObserving.length;
        while (l--) {
            const dep = prevObserving[l];
            if (dep.diffValue === 0) {
                removeObserver(dep, derivation);
            }
            dep.diffValue = 0;
        }
    
        while (i0--) {
            const dep = observing[i0];
            if (dep.diffValue === 1) {
                dep.diffValue = 0;
                addObserver(dep, derivation);
            }
        }
    
        // derivation.newObserving下某数组项observable的dependenciesState为-1时
        // lowestNewObservingDerivationState赋值为-1
        // derivation.dependenciesState置为-1，并执行derivation.onBecomeStale方法
        if (lowestNewObservingDerivationState !== IDerivationState.UP_TO_DATE) {
            derivation.dependenciesState = lowestNewObservingDerivationState;
            derivation.onBecomeStale();
        }
    }
    
    // clearObserving(derivation)，重置derivation
    // 清空derivation.observing数组，derivation.observing=[IObservable]数组项observable实例移除derivation
    // 并将derivation.dependenciesState置为-1
    export function clearObserving(derivation: IDerivation) {
        const obs = derivation.observing;
        derivation.observing = [];
        let i = obs.length;
        while (i--) removeObserver(obs[i], derivation);
    
        derivation.dependenciesState = IDerivationState.NOT_TRACKING;
    }
    
    // untracked(action)
    // 执行action，执行前后缓存globalState.trackingDerivation前一个值，并在action执行结束后回设该值
    export function untracked<T>(action: () => T): T {
        const prev = untrackedStart();
        const res = action();
        untrackedEnd(prev);
        return res;
    }
    
    // untrackedStart()
    // 取出globalState.trackingDerivation，且将globalState.trackingDerivation置为null
    export function untrackedStart(): IDerivation | null {
        const prev = globalState.trackingDerivation;
        globalState.trackingDerivation = null;
        return prev;
    }
    
    // untrackedEnd(prev)，将globalState.trackingDerivation设为prev
    // prev通常是untrackedStart函数取出的globalState.trackingDerivation，即前一个值
    export function untrackedEnd(prev: IDerivation | null) {
        globalState.trackingDerivation = prev;
    }
    
    // changeDependenciesStateTo0(derivation)
    // 将derivation.dependenciesState置为0，derivation.observing=[IObservable]数组项的lowestObserverState置为0
    export function changeDependenciesStateTo0(derivation: IDerivation) {
        if (derivation.dependenciesState === IDerivationState.UP_TO_DATE) return;
        derivation.dependenciesState = IDerivationState.UP_TO_DATE;
    
        const obs = derivation.observing;
        let i = obs.length;
        while (i--) obs[i].lowestObserverState = IDerivationState.UP_TO_DATE;
    }
