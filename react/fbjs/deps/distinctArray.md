# distinctArray

distinctArray(arr)，通过arr类型将xs数组去重后返回。

    'use strict';

    var Set = require('./Set');

    // 通过Set类型将xs数组去重后返回
    function distinctArray(xs) {
      return Array.from(new Set(xs).values());
    }

    module.exports = distinctArray;