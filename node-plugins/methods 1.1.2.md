# methods 1.1.2

node支持的http请求方式。

    'use strict';
    
    var http = require('http');
    
    module.exports = getCurrentNodeMethods() || getBasicNodeMethods();
    
    function getCurrentNodeMethods() {
      return http.METHODS && http.METHODS.map(function lowerCaseMethod(method) {
        return method.toLowerCase();
      });
    }
    
    function getBasicNodeMethods() {
      return [
        'get',
        'post',
        'put',
        'head',
        'delete',
        'options',
        'trace',
        'copy',
        'lock',
        'mkcol',
        'move',
        'purge',
        'propfind',
        'proppatch',
        'unlock',
        'report',
        'mkactivity',
        'checkout',
        'merge',
        'm-search',
        'notify',
        'subscribe',
        'unsubscribe',
        'patch',
        'search',
        'connect'
      ];
    }
