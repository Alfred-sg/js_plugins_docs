# isomorphic-fetch 2.2.1

## 概述

fetch(url, options)
调用node-fetch模块实现node端发送请求；调用whatwg-fetch实现客户端发送ajax请求。

## 源码

### package.json

    "browser": "fetch-npm-browserify.js",
    "main": "fetch-npm-node.js"

### fetch-npm-browserify.js

    require('whatwg-fetch');
    module.exports = self.fetch.bind(self);

### fetch-npm-node.js 

    "use strict";
    
    var realFetch = require('node-fetch');
    module.exports = function(url, options) {
        if (/^\/\//.test(url)) {
            url = 'https:' + url;
        }
        return realFetch.call(this, url, options);
    };
    
    if (!global.fetch) {
        global.fetch = module.exports;
        global.Response = realFetch.Response;
        global.Headers = realFetch.Headers;
        global.Request = realFetch.Request;
    }
