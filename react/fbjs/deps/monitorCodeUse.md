# monitorCodeUse

monitorCodeUse(eventName,data)，检测eventName只包含[^a-z0-9_]字符。

    'use strict';

    var invariant = require('./invariant');

    // 检测eventName只包含[^a-z0-9_]字符
    function monitorCodeUse(eventName, data) {
      !(eventName && !/[^a-z0-9_]/.test(eventName)) ? process.env.NODE_ENV !== 'production' ? 
        invariant(false, 'You must provide an eventName using only the characters [a-z0-9_]') 
        : invariant(false) : void 0;
    }

    module.exports = monitorCodeUse;