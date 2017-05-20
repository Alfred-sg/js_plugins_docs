# Set

利用'core-js'模块构建Set类型。

    'use strict';

    // 利用'core-js'模块构建Set类型，即成员唯一的数组；相比WeakSet，WeakSet成员只能是对象，且对象为弱引用
    // 
    // new Set(arr) 以数组形式初始化
    // 
    // Set类型原型方法
    // constructor()返回构造函数，默认为Set
    // size()返回成员总数
    // add(value)添加value，返回Set实例
    // delete(value)删除value，返回删除成功与否的布尔值
    // has(value)判断value是否在set实例中
    // clear()清空set实例，没有返回值
    // keys()返回迭代器，遍历键名数组[key]，即返回值调用next方法得到set实例成员数组[value]
    // values()返回迭代器，遍历键值数组[value]，即返回值调用next方法得到set实例成员数组[value]
    //      且Set.prototype[Symbol.iterator]=Set.prototype.values
    // entries()返回迭代器，遍历键值对数组[key,value]，即返回值调用next方法得到set实例成员数组[value,value]
    // forEach(fn=(value,key)=>{})遍历成员，执行fn函数，没有返回值
    module.exports = require('core-js/library/es6/set');