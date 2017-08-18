# spy.md

## 概述

类同redux中间件，侦测变动数据，可用于打印日志等。

## api

* isSpyEnabled()，判断globalState是否有spyListeners绑定函数。
* spyReport(event)，以event参数执行globalState.spyListeners中各绑定函数。
* spyReportStart(event)，以{ ...event, spyReportStart: true }执行globalState.spyListeners中各绑定函数。
* spyReportEnd(change?)，以{ ...change, spyReportEnd: true }执行globalState.spyListeners中各绑定函数。
* spy(listener)，向globalState.spyListeners添加绑定函数listener，返回函数用于移除该绑定函数。

## 源码

    import {
        globalState// 全局缓存globalState
    } from "./globalstate";
    import {
        objectAssign,// 浅拷贝
        once,// once(func)，使func执行一次
        Lambda// 无值型函数接口
    } from "../utils/utils";
    
    // isSpyEnabled()，判断globalState是否有spyListeners绑定函数
    export function isSpyEnabled() {
        return !!globalState.spyListeners.length;
    }
    
    // spyReport(event)，以event参数执行globalState.spyListeners中各绑定函数
    export function spyReport(event) {
        if (!globalState.spyListeners.length) return;
        const listeners = globalState.spyListeners;
        for (let i = 0, l = listeners.length; i < l; i++)
            listeners[i](event);
    }
    
    // spyReportStart(event)，以{ ...event, spyReportStart: true }执行globalState.spyListeners中各绑定函数
    export function spyReportStart(event) {
        const change = objectAssign({}, event, { spyReportStart: true });
        spyReport(change);
    }
    
    const END_EVENT = { spyReportEnd: true };
    
    // spyReportEnd(change?)，以{ ...change, spyReportEnd: true }执行globalState.spyListeners中各绑定函数
    export function spyReportEnd(change?) {
        if (change) spyReport(objectAssign({}, change, END_EVENT));
        else spyReport(END_EVENT);
    }
    
    // spy(listener)，向globalState.spyListeners添加绑定函数listener，返回函数用于移除该绑定函数
    export function spy(listener: (change: any) => void): Lambda {
        globalState.spyListeners.push(listener);
        return once(() => {
            const idx = globalState.spyListeners.indexOf(listener);
            if (idx !== -1)
                globalState.spyListeners.splice(idx, 1);
        });
    }
