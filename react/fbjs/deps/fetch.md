# fetch

fetch(url, options)，调用node-fetch模块实现node端发送请求；调用whatwg-fetch实现客户端发送ajax请求。

    'use strict';
    
    if (global.fetch) {
      module.exports = global.fetch.bind(global);
    } else {
      // fetch(url, options)，调用node-fetch模块实现node端发送请求；调用whatwg-fetch实现客户端发送ajax请求
      module.exports = require('isomorphic-fetch');
    }