# koa-static 2.0.0 

## 概述

根据请求地址this.path发送静态资源。

## 使用

    var serve = require('koa-static');
    var koa = require('koa');
    var app = koa();
    
    // $ GET /package.json
    app.use(serve('.'));
    
    // $ GET /hello.txt
    app.use(serve('test/fixtures'));
    
    // or use absolute paths
    app.use(serve(__dirname + '/test/fixtures'));
    
    app.listen(3000);
    
    console.log('listening on port 3000');

## 源码

    'use strict';
    
    const resolve = require('path').resolve;
    const assert = require('assert');
    const debug = require('debug')('koa-static');
    const send = require('koa-send');
    
    module.exports = serve;
    
    // 根据请求地址this.path发送静态资源
    function serve(root, opts) {
      opts = opts || {};
    
      assert(root, 'root directory is required to serve files');
    
      debug('static "%s" %j', root, opts);
      opts.root = resolve(root);
      if (opts.index !== false) opts.index = opts.index || 'index.html';
    
      if (!opts.defer) {
        // 先发送静态文件，再执行后续中间件
        return function *serve(next){
          if (this.method == 'HEAD' || this.method == 'GET') {
            if (yield send(this, this.path, opts)) return;
          }
          yield* next;
        };
      }
    
      // 先执行后续中间件，再发送静态文件
      return function *serve(next){
        yield* next;
    
        if (this.method != 'HEAD' && this.method != 'GET') return;
        if (this.body != null || this.status != 404) return;
    
        yield send(this, this.path, opts);
      };
    }
