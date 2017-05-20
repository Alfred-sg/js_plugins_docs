# promise.native

new Promise((resolve,reject)=>{})，调用promise模块构建延迟对象，只包含浏览器端方法。

### 静态方法

* Promise.resolve(arg=[null|undefined|true|false|0|''|{then:()=>{}}|...])构建promise实例的工厂函数，以参数arg作为成功回调的参数；当arg为promise实例或thenable对象时，以promise模块机制等待延迟，将延迟结果作为成功回调的参数。
* Promise.reject(value)构建promise实例的工厂函数，以参数value作为失败回调的参数。
* Promise.all(arr=[{then:()=>{}}|...])构建promise实例的工厂函数，参数arr中含由promise实例或thenable对象时，等待延迟，以延迟结果构建成功、失败回调的参数数组；否则直接将arr作为成功、失败回调的参数。
* Promise.race(arr=[{then:()=>{}}|...])构建promise实例的工厂函数，参数arr中含由promise实例或thenable对象时，每个延迟结束即调用成功、失败回调；否则以数组arr长度执行多次成功、失败回调。

### 原型方法

* promise.then(onFulfilled,onRejected)设定成功、失败回调。
* promise.done(onFulfilled,onRejected)同then方法，设定成功、失败回调；遇错时报错。
* promise.finally(fn=()=>{})设定成功、失败回调为fn。
* promise.catch(onRejected)设定失败回调。

'use strict';

var Promise = require('promise/setimmediate/es6-extensions');
require('promise/setimmediate/done');

Promise.prototype['finally'] = function (onSettled) {
  return this.then(onSettled, onSettled);
};

module.exports = Promise;
