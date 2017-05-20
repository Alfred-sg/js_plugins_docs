# promise

new Promise((resolve,reject)=>{})，调用promise模块构建延迟对象。

thenable对象--带then方法的对象。
* new Promise(fn=(resolve,reject)=>{})创建promise实例，参数fn调用resolve、reject函数设定成功、失败回调的执行时机；resolve、reject均由promise模块机制传入fn中。

### 静态方法

* Promise.resolve(arg=[null|undefined|true|false|0|''|{then:()=>{}}|...])构建promise实例的工厂函数，以参数arg作为成功回调的参数；当arg为promise实例或thenable对象时，以promise模块机制等待延迟，将延迟结果作为成功回调的参数。
* Promise.reject(value)构建promise实例的工厂函数，以参数value作为失败回调的参数。
* Promise.all(arr=[{then:()=>{}}|...])构建promise实例的工厂函数，参数arr中含由promise实例或thenable对象时，等待延迟，以延迟结果构建成功、失败回调的参数数组；否则直接将arr作为成功、失败回调的参数。
* Promise.race(arr=[{then:()=>{}}|...])构建promise实例的工厂函数，参数arr中含由promise实例或thenable对象时，每个延迟结束即调用成功、失败回调；否则以数组arr长度执行多次成功、失败回调。
* Promise.denodeify(fn,num)用于生成构建promise实例的工厂函数，fn尾参以函数形式(err, res)=>{}设定成功、失败执行时机，err为真值时执行失败回调、为否值时执行成功回调；默认情况下，fn返回promise实例或thenable对象，等待延迟，以延迟结果作为成功回调参数；否则直接将fn返回值作为成功回调函数。
* Promise.nodeify(fn)返回函数(...args,[callback])=>{}，调用时将执行fn(args)，若fn返回promise对象，等待延迟，最终执行callback回调；若fn返回普通值，且callback为否值，执行错误回调；若fn返回普通纸，且callback为真值，执行callback。
* Promise.enableSynchronous()激活isPending、isFulfilled、isRejected、getValue、getReason、getState原型方法功能。
* Promise.enableSynchronous()关闭isPending、isFulfilled、isRejected、getValue、getReason、getState原型方法功能。

### 原型方法

* promise.then(onFulfilled,onRejected)设定成功、失败回调。
* promise.done(onFulfilled,onRejected)同then方法，设定成功、失败回调；遇错时报错。
* promise.finally(fn=()=>{})通过参数fn函数将内部缓存self._65设定为fn的返回值，再次调用then、done方法添加的成功、失败回调函数启用执行时，都将以fn返回值作为参数。_
* promise.catch(onRejected)设定失败回调。
* promise.nodify(callback,ctx)设定成功、失败函数为以上下文ctx执行callback。
* promise.isPending()判断异步函数是否执行中。
* promise.isFulfilled()判断异步函数是否执行成功。
* promise.isRejected()判断异步函数是否执行失败。
* promise.getValue()判断异步函数的执行结果；异步函数返回promise实例，递归取该promise实例的延迟执行结果。
* promise.getReason()判断异步函数的执行结果；异步函数返回promise实例，递归取该promise实例的延迟执行结果。
* promise.isRejected()获取异步函数执行状态，0-pending，1-resovled，2-rejected；异步函数返回promise实例，递归取该promise的延迟函数执行状态。

    'use strict';

    module.exports = require('promise');
