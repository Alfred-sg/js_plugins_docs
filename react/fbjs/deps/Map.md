# Map

new Map()，利用'core-js'模块构建Map类型。

    'use strict';

    // 利用'core-js'模块构建Map类型；相比对象，Map类型的键可以是任何类型
    // 
    // new Map();
    // 
    // 原型方法
    // size()返回成员总数
    // set(key,value)设置某属性
    // get(key)获取某属性
    // has(key)判断是否存在某属性
    // delete(key)删除某属性
    // clear()清除所有成员
    // keys()返回所有键名的迭代器
    // values()返回所有键值的迭代器
    // entries()返回所有键名、键值的迭代器
    // forEach(fn=(value,key)=>{})遍历所有成员，并执行fn函数
    module.exports = require('core-js/library/es6/map');