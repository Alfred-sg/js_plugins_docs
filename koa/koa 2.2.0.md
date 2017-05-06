# koa 2.2.0

## 概述

koa以中间件形式构建http|https.createServer(fn(req,res))的回调函数fn(req,res)；同时将封装请求req、响应res为request对象和response对象，前者赋予解析路径、判断请求头的能力，后者赋予设置响应、响应头的能力；最终将request对象和response对象作为各中间件上下文对象ctx的属性，ctx又能代理访问request对象和response对象的属性和方法，ctx还拥有一定的错误处理能力，用于抛出错误、捕获错误、将错误作为响应。

## 使用

### 示例1

	var koa = require('koa');
	var app = koa();
	
	// 2.2.0版本中间件可挂载生成器函数、普通函数、async函数
	app.use(async function (){
	  this.body = 'Hello World';
	});
	
	app.listen(3000);
	
### 示例2，多端口监听同一应用

	var http = require('http');
	var koa = require('koa');
	var app = koa();
	http.createServer(app.callback()).listen(3000);
	http.createServer(app.callback()).listen(3001);
	
## 核心源码

### application.js

use方法挂载中间件；createContext方法生成中间件执行上下文对象ctx；callback方法将中间串联为一，注入上下文ctx；listen方法启动http服务。

	'use strict';
	
	const isGeneratorFunction = require('is-generator-function');// 判断是否生成器函数
	const debug = require('debug')('koa:application');// 打印日志
	const onFinished = require('on-finished');// onFinish(res,fn)响应终止时执行回调fn
	const response = require('./response');
	const compose = require('koa-compose');// 串联中间件，返回promise对象
	const isJSON = require('koa-is-json');// 判断变量是否需要被识别json对象加以处理
	const context = require('./context');
	const request = require('./request');
	const statuses = require('statuses');// http状态码类型
	const Cookies = require('cookies');// 以this.keys设置cookies签名，以识别cookies是否被篡改
	const accepts = require('accepts');// 校验请求头的accept属性，客户端对响应的偏好
	const Emitter = require('events');// 事件模块
	const assert = require('assert');// 报错
	const Stream = require('stream');// 流模块
	const http = require('http');// 默认启用http服务
	const only = require('only');// 类同es6解构赋值
	const convert = require('koa-convert');// 用于转换生成器函数
	const deprecate = require('depd')('koa');// 移除的方法提示
	
	// 创建koa应用，该引用继承events模块，可通过app=koa();app.on("error",fn)绑定事件
	// app.use方法挂载中间件
	// app.callback方法将中间件串联为一，构成fn(req,res)函数，作为http|https.createServer的回调
	//    默认创建http服务；借助https.createServer(app.callback()).listen(port)创建https服务
	module.exports = class Application extends Emitter {
	
	  constructor() {
	    super();// 事件对象初始化
	
	    this.proxy = false;// 是否http代理或负载均衡技术实现代理；请求头中含有'X-Forwarded-For'属性包含原始和各代理ip地址
	    this.middleware = [];// 存取中间件
	    this.subdomainOffset = 2;// 获取子域名的起始位；如"tobi.ferrets.example.com"，子域名为["ferrets", "tobi"]，主域名为["com","example"]
	    this.env = process.env.NODE_ENV || 'development';// 用以区别开发环境
	    this.context = Object.create(context);// 中间件this.context调用
	    this.request = Object.create(request);// 中间件this.request调用
	    this.response = Object.create(response);// 中间件this.response调用
	  }
	
	  // 创建http服务
	  listen() {
	    debug('listen');
	    const server = http.createServer(this.callback());
	    return server.listen.apply(server, arguments);
	  }
	
	  // 等同es2015解构，获取this关键字的{subdomainOffset,proxy,env}属性集合
	  toJSON() {
	    return only(this, [
	      'subdomainOffset',
	      'proxy',
	      'env'
	    ]);
	  }
	
	  inspect() {
	    return this.toJSON();
	  }
	
	  // 中间件可使用普通函数或async函数(node v7.6+)书写；当前版本仍支持生成器函数
	
	  // app.use(async (ctx, next) => {
	  //   const start = new Date();
	  //   await next();
	  //   const ms = new Date() - start;
	  //   console.log(`${ctx.method} ${ctx.url} - ${ms}ms`);
	  // });
	
	  // app.use((ctx, next) => {
	  //   const start = new Date();
	  //   return next().then(() => {
	  //     const ms = new Date() - start;
	  //     console.log(`${ctx.method} ${ctx.url} - ${ms}ms`);
	  //   });
	  // });
	
	  // 挂载中间件fn(ctx,next)或async fn(ctx,next)，或fn *(next)
	  use(fn) {
	    if (typeof fn !== 'function') throw new TypeError('middleware must be a function!');
	    if (isGeneratorFunction(fn)) {
	      deprecate('Support for generators will be removed in v3. ' +
	                'See the documentation for examples of how to convert old middleware ' +
	                'https://github.com/koajs/koa/blob/master/docs/migration.md');
	
	      // 转化生成器函数fn，使用co模块调用生成器函数，促使其返回promise对象，生成器函数调用时以ctx作为上下文对象
	      // 同时将koa注入的next下一个中间件即下一个生成器函数，作为参数传入生成器函数fn中
	      fn = convert(fn);
	    }
	    debug('use %s', fn._name || fn.name || '-');
	    this.middleware.push(fn);
	    return this;
	  }
	
	  // 将添加的中间件串联为一，改造成fn(req,res)形式，为中间件函数提供ctx对象，并返回
	  callback() {
	    const fn = compose(this.middleware);
	
	    if (!this.listeners('error').length) this.on('error', this.onerror);
	
	    const handleRequest = (req, res) => {
	      res.statusCode = 404;
	      const ctx = this.createContext(req, res);// 生成中间件的上下文this即ctx
	      const onerror = err => ctx.onerror(err);
	      const handleResponse = () => respond(ctx);
	      onFinished(res, onerror);
	
	      // 使用promise对象的then挂载handleResponse响应处理函数，catch方法挂载onerror错误处理函数
	      return fn(ctx).then(handleResponse).catch(onerror);
	    };
	
	    return handleRequest;
	  }
	
	  // 生成中间件的上下文对象ctx，{request,response,app,req,res,ctx,originalUrl,cookies,accept,state}形式
	  // 实际创建ctx的过程中，req、res属性基于http|https.createServer(fn(req,res))中参数req、res注入生成
	  createContext(req, res) {
	    const context = Object.create(this.context);
	
	    // 封装http|https.createServer(fn(req,res))的参数req，提供解析获取url查询字符串、判断请求头等功能
	    const request = context.request = Object.create(this.request);
	
	    // 封装http|https.createServer(fn(req,res))的参数res，设置响应内容、状态码、响应头等功能
	    const response = context.response = Object.create(this.response);
	
	    context.app = request.app = response.app = this;
	
	    context.req = request.req = response.req = req;// 指向http|https.createServer(fn(req,res))的参数req
	    context.res = request.res = response.res = res;// 指向http|https.createServer(fn(req,res))的参数res
	    request.ctx = response.ctx = context;
	    request.response = response;
	    response.request = request;
	    context.originalUrl = request.originalUrl = req.url;// 原始的请求地址
	    
	    // 用于校验请求头的cookie、设置响应头的cookie
	    // this.cookies.get(name,[{signed:true}]) 无signed直接取值
	    // 		有signed，比较name+".sig"是否匹配this.keys加密后的密文，不匹配，无返回值，匹配，取name的值
	    // this.cookies.set(name,value,[{signed:true}]) signed为真，同时发送name+".sig"密文
	    context.cookies = new Cookies(req, res, {// 设置cookies签名
	      keys: this.keys,
	      secure: request.secure
	    });
	    request.ip = request.ips[0] || req.socket.remoteAddress || '';// 请求的原始id地址
	    context.accept = request.accept = accepts(req);// 用以校验请求头的accept属性，以获悉客户端偏好何种响应
	    context.state = {};
	    return context;
	  }
	
	  // 默认的错误处理函数；err.status为404或err.expose为真或this.silent为真时，略过报错处理
	  onerror(err) {
	    assert(err instanceof Error, `non-error thrown: ${err}`);
	
	    if (404 == err.status || err.expose) return;
	    if (this.silent) return;
	
	    const msg = err.stack || err.toString();
	    console.error();
	    console.error(msg.replace(/^/gm, '  '));
	    console.error();
	  }
	};
	
	// 默认的响应处理功能
	function respond(ctx) {
	  // ctx.respond设置为否时，禁用KOA内置的响应处理功能
	  if (false === ctx.respond) return;
	
	  const res = ctx.res;
	  if (!ctx.writable) return;// ctx.writable即this.response.writable是否可设置响应
	
	  let body = ctx.body;// ctx.body;即this.response.body响应内容
	  const code = ctx.status;// ctx.status即this.response.status状态码
	
	  // ctx.status为204、205、304时，响应为空
	  if (statuses.empty[code]) {
	    ctx.body = null;
	    return res.end();
	  }
	
	  if ('HEAD' == ctx.method) {
	    if (!res.headersSent && isJSON(body)) {// res.headersSent响应头是否经socket形式发出，用于在发生错误时检查客户端是否被通知
	      ctx.length = Buffer.byteLength(JSON.stringify(body));
	    }
	    return res.end();
	  }
	
	  // status body
	  if (null == body) {
	    body = ctx.message || String(code);
	    if (!res.headersSent) {
	      ctx.type = 'text';
	      ctx.length = Buffer.byteLength(body);
	    }
	    return res.end(body);
	  }
	
	  // 根据body类型Buffer、字符串、文件流、json对象，输出相应
	  if (Buffer.isBuffer(body)) return res.end(body);
	  if ('string' == typeof body) return res.end(body);
	  if (body instanceof Stream) return body.pipe(res);
	
	  body = JSON.stringify(body);
	  if (!res.headersSent) {
	    ctx.length = Buffer.byteLength(body);
	  }
	  res.end(body);
	}

### request.js

封装请求，解析请求路径、判断请求头。

	'use strict';
	
	const net = require('net');
	
	// parse方法以对象形式解析媒体文件类型MIME，format方法将对象形式MIME转化为字符串
	const contentType = require('content-type');
	
	// 将{protocolis,hostname,port,host,pathname,query,search,hash}解析为路径字符串
	const stringify = require('url').format;
	
	// 将req.url或req.originalUrl解析为对象形式
	const parse = require('parseurl');
	
	// parse方法解析url携带参数，stringify方法将对象形式的url参数转化为字符串形式
	const qs = require('querystring');
	
	// 判断请求req的MIME媒体文件类型是否和传参MIME类型匹配
	const typeis = require('type-is');
	
	// 通过请求头校验请求是否被更新
	const fresh = require('fresh');
	
	const only = require('only');
	
	// 封装http|https.createServer(fn(req,res))的参数req，提供解析获取url查询字符串、判断请求头等功能
	module.exports = {
	
	  // 获取请求头
	  get header() {
	    return this.req.headers;
	  },
	
	  // 获取请求头
	  get headers() {
	    return this.req.headers;
	  },
	
	  // 获取请求地址
	  get url() {
	    return this.req.url;
	  },
	
	  // 设置请求地址
	  set url(val) {
	    this.req.url = val;
	  },
	
	  // originalUrl 原始的请求地址，application模块的createContext方法中赋值
	
	  // 获取协议名和域名，"http://example.com"形式
	  get origin() {
	    return `${this.protocol}://${this.host}`;
	  },
	
	  // 获取原始请求地址的全名，"http://example.com/foo"形式
	  get href() {
	    if (/^https?:\/\//i.test(this.originalUrl)) return this.originalUrl;
	    return this.origin + this.originalUrl;
	  },
	
	  // 获取请求方法
	  get method() {
	    return this.req.method;
	  },
	
	  // 设置请求方法
	  set method(val) {
	    this.req.method = val;
	  },
	
	  // 获取请求路径名，不含域名、参数等信息
	  get path() {
	    return parse(this.req).pathname;
	  },
	
	  // 设置请求路径；将完整路径赋值给this.url，含参数和hash值
	  set path(path) {
	    const url = parse(this.req);
	    if (url.pathname === path) return;
	
	    url.pathname = path;
	    url.path = null;
	
	    this.url = stringify(url);
	  },
	
	  // 以对象形式获取请求路径中携带的参数
	  get query() {
	    const str = this.querystring;
	    const c = this._querycache = this._querycache || {};
	    return c[str] || (c[str] = qs.parse(str));
	  },
	
	  // 以对象形式设置请求路径的参数；将完整路径赋值给this.url，含参数和hash值
	  set query(obj) {
	    this.querystring = qs.stringify(obj);
	  },
	
	  // 以字符串形式获取请求路径中携带的参数，不带"?"
	  get querystring() {
	    if (!this.req) return '';
	    return parse(this.req).query || '';
	  },
	
	  // 以字符串形式设置请求路径的参数；将完整路径赋值给this.url，含参数和hash值
	  set querystring(str) {
	    const url = parse(this.req);
	    if (url.search === `?${str}`) return;
	
	    url.search = str;
	    url.path = null;
	
	    this.url = stringify(url);
	  },
	
	  // 以字符串形式获取请求路径中携带的参数，带"?"
	  get search() {
	    if (!this.querystring) return '';
	    return `?${this.querystring}`;
	  },
	
	  // 以字符串形式设置请求路径的参数；将完整路径赋值给this.url，含参数和hash值
	  set search(str) {
	    this.querystring = str;
	  },
	
	  // 获取域名(含端口)，即ip地址
	  // 若this.app.proxy为真，支持通过this.get('X-Forwarded-Host')获取使用http代理或负载均衡前的原始ip地址
	  get host() {
	    const proxy = this.app.proxy;
	    let host = proxy && this.get('X-Forwarded-Host');
	    host = host || this.get('Host');
	    if (!host) return '';
	    return host.split(/\s*,\s*/)[0];
	  },
	
	  // 获取域名，不含端口
	  get hostname() {
	    const host = this.host;
	    if (!host) return '';
	    return host.split(':')[0];
	  },
	
	  // 校验请求是否被更新
	  get fresh() {
	    const method = this.method;
	    const s = this.ctx.status;
	
	    // GET or HEAD for weak freshness validation only
	    if ('GET' != method && 'HEAD' != method) return false;
	
	    // 2xx or 304 as per rfc2616 14.26
	    if ((s >= 200 && s < 300) || 304 == s) {
	      return fresh(this.header, this.ctx.response.header);
	    }
	
	    return false;
	  },
	
	  get stale() {
	    return !this.fresh;
	  },
	
	  // 校验请求方法是否为'GET','HEAD','PUT','DELETE','OPTIONS','TRACE'中的一种
	  get idempotent() {
	    const methods = ['GET', 'HEAD', 'PUT', 'DELETE', 'OPTIONS', 'TRACE'];
	    return !!~methods.indexOf(this.method);
	  },
	
	  // 获取请求socket
	  get socket() {
	    return this.req.socket;
	  },
	
	  // 获取'Content-Type'，互联网媒体类型，MIME类型，定义网页的文件类型和编码，决定浏览器的读取方式
	  // 类型格式：type/subtype(;parameter)? 
	  // type 主类型，任意的字符串，如text，如果是*号代表所有；   
	  // subtype 子类型，任意的字符串，如html，如果是*号代表所有；   
	  // parameter 可选，一些参数，如Accept请求头的q参数，Content-Type的charset参数。 
	  // 获取MIME类型参数中设置的编码方式
	  get charset() {
	    let type = this.get('Content-Type');
	    if (!type) return '';
	
	    try {
	      type = contentType.parse(type);
	    } catch (e) {
	      return '';
	    }
	
	    return type.parameters.charset || '';
	  },
	
	  // 以数字形式获取请求的内容长度Content-Length
	  get length() {
	    const len = this.get('Content-Length');
	    if (len == '') return;
	    return ~~len;// ~~等同Math.floor
	  },
	
	  // 获取协议名http或https
	  get protocol() {
	    const proxy = this.app.proxy;
	    if (this.socket.encrypted) return 'https';
	    if (!proxy) return 'http';
	    const proto = this.get('X-Forwarded-Proto') || 'http';
	    return proto.split(/\s*,\s*/)[0];
	  },
	
	  // 是否安全性更高的https协议
	  get secure() {
	    return 'https' == this.protocol;
	  },
	
	  // 通过http代理或负载均衡技术实现代理，以数组形式获取原始ip地址和各代理地址
	  get ips() {
	    const proxy = this.app.proxy;
	    const val = this.get('X-Forwarded-For');
	    return proxy && val
	      ? val.split(/\s*,\s*/)
	      : [];
	  },
	
	  // 根据this.app.subdomainOffset获取子域名
	  // 如"tobi.ferrets.example.com"，默认取子域名为["ferrets", "tobi"]，主域名为["com","example"]
	  get subdomains() {
	    const offset = this.app.subdomainOffset;
	    const hostname = this.hostname;
	    if (net.isIP(hostname)) return [];
	    return hostname
	      .split('.')
	      .reverse()
	      .slice(offset);
	  },
	
	  // 请求头中的accept属性，设定客户端偏好何种响应
	  // 校验请求头中'accept'属性即客户端偏好响应类型是否匹配传参数组中的任意项；若匹配返回匹配值
	  // Accept: text/*, application/json
	  // this.accepts('html'); // => "html"
	  // this.accepts('text/html'); // => "text/html"
	  // this.accepts('json', 'text'); // => "json"
	  // this.accepts('application/json'); // => "application/json"
	  // this.accepts('image/png'); // => false
	  accepts() {
	    return this.accept.types.apply(this.accept, arguments);
	  },
	
	  // 校验请求头中'accept-encoding'属性是否匹配传参数组中的任意项；若匹配返回匹配值
	  // 参数如['gzip', 'deflate', 'identity']
	  acceptsEncodings() {
	    return this.accept.encodings.apply(this.accept, arguments);
	  },
	
	  // 校验请求头中'accept-charset'属性是否匹配传参数组中的任意项；若匹配返回匹配值
	  // 参数如['utf-8', 'utf-7', 'iso-8859-1']
	  acceptsCharsets() {
	    return this.accept.charsets.apply(this.accept, arguments);
	  },
	
	  // 校验请求头中'accept-language'属性是否匹配传参数组中的任意项；若匹配返回匹配值
	  // 参数如['es', 'pt', 'en']
	  acceptsLanguages() {
	    return this.accept.languages.apply(this.accept, arguments);
	  },
	
	  // 校验请求头中'Content-Type'属性即MIME文件类型是否匹配传参数组中的任意项；若匹配返回匹配值
	  // When Content-Type is application/json
	  // this.is('json', 'urlencoded'); // => 'json'
	  // this.is('application/json'); // => 'application/json'
	  // this.is('html', 'application/*'); // => 'application/json'
	  // this.is('html'); // => false
	  is(types) {
	    if (!types) return typeis(this.req);
	    if (!Array.isArray(types)) types = [].slice.call(arguments);
	    return typeis(this.req, types);
	  },
	
	  // 获取请求头中'Content-Type'属性即MIME文件类型，不包含参数
	  get type() {
	    const type = this.get('Content-Type');
	    if (!type) return '';
	    return type.split(';')[0];
	  },
	
	  // 获取请求头中的field属性
	  get(field) {
	    const req = this.req;
	    switch (field = field.toLowerCase()) {
	      case 'referer':
	      case 'referrer':
	        return req.headers.referrer || req.headers.referer || '';
	      default:
	        return req.headers[field] || '';
	    }
	  },
	
	  // 获取请求方法、请求路径、请求头{method,url,header}属性集合
	  inspect() {
	    if (!this.req) return;
	    return this.toJSON();
	  },
	
	  // 获取请求方法、请求路径、请求头{method,url,header}属性集合
	  toJSON() {
	    return only(this, [
	      'method',
	      'url',
	      'header'
	    ]);
	  }
	};

### response.js

封装响应，设置响应内容、状态码、响应头。
	
	'use strict';
	
	const contentDisposition = require('content-disposition');// 通过文件名获取响应头`Content-Disposition`属性
	const ensureErrorHandler = require('error-inject');// onFinish(stream,fn)流操作报错时执行回调fn
	const getType = require('mime-types').contentType;// 获取mime
	const onFinish = require('on-finished');// onFinish(res,fn)响应终止时执行回调fn
	const isJSON = require('koa-is-json');// 是否需要作为json数据处理
	const escape = require('escape-html');// http转义
	const typeis = require('type-is').is;// 判断请求req的MIME媒体文件类型是否和传参MIME类型匹配
	const statuses = require('statuses');// http状态码类型
	const destroy = require('destroy');// 用于销毁流
	const assert = require('assert');// 报错
	const extname = require('path').extname;// 获取文件扩展名
	const vary = require('vary');// 设置响应头的vary属性
	const only = require('only');// 解构赋值
	
	module.exports = {
	
	  // 获取请求的socket
	  get socket() {
	    return this.ctx.req.socket;
	  },
	
	  // 通过http|https.createServer(fn(req,res))的参数res获取响应头
	  get header() {
	    const { res } = this;
	    return typeof res.getHeaders === 'function'
	      ? res.getHeaders()
	      : res._headers || {};  // Node < 7.7
	  },
	
	  // 获取响应头
	  get headers() {
	    return this.header;
	  },
	
	  // 通过http|https.createServer(fn(req,res))的参数res获取statusCode
	  get status() {
	    return this.res.statusCode;
	  },
	
	  // 设置http状态码statusCode及状态码文案statusMessage；204、205、304状态码，设置this.body为null
	  set status(code) {
	    assert('number' == typeof code, 'status code must be a number');
	    assert(statuses[code], `invalid status code: ${code}`);
	    assert(!this.res.headersSent, 'headers have already been sent');
	    this._explicitStatus = true;// 跳过自动设置this.status状态码
	    this.res.statusCode = code;
	    this.res.statusMessage = statuses[code];
	    if (this.body && statuses.empty[code]) this.body = null;
	  },
	
	  // 获取http状态码文案
	  get message() {
	    return this.res.statusMessage || statuses[this.status];
	  },
	
	  // 设置http状态码文案
	  set message(msg) {
	    this.res.statusMessage = msg;
	  },
	
	  // 获取响应this._body
	  get body() {
	    return this._body;
	  },
	
	  // String|Buffer|Object|Stream形式设置响应this._body
	  // 无内容，状态码自动设为204；未经this.status状态码赋值，自动设为200状态码
	  set body(val) {
	    const original = this._body;
	    this._body = val;
	
	    if (this.res.headersSent) return;
	
	    // 无内容，状态码自动设为204
	    if (null == val) {
	      if (!statuses.empty[this.status]) this.status = 204;
	      this.remove('Content-Type');
	      this.remove('Content-Length');
	      this.remove('Transfer-Encoding');
	      return;
	    }
	
	    // this._explicitStatus为否，未经this.status状态码赋值，自动设为200状态码
	    if (!this._explicitStatus) this.status = 200;
	
	    // 响应头'content-type'属性未设置，自动赋值
	    const setType = !this.header['content-type'];
	
	    // string
	    if ('string' == typeof val) {
	      if (setType) this.type = /^\s*</.test(val) ? 'html' : 'text';
	      this.length = Buffer.byteLength(val);
	      return;
	    }
	
	    // buffer
	    if (Buffer.isBuffer(val)) {
	      if (setType) this.type = 'bin';
	      this.length = val.length;
	      return;
	    }
	
	    // stream
	    if ('function' == typeof val.pipe) {
	      onFinish(this.res, destroy.bind(null, val));
	      ensureErrorHandler(val, err => this.ctx.onerror(err));
	
	      // overwriting
	      if (null != original && original != val) this.remove('Content-Length');
	
	      if (setType) this.type = 'bin';
	      return;
	    }
	
	    // json
	    this.remove('Content-Length');
	    this.type = 'json';
	  },
	
	  // 以数字形式设置响应的内容长度Content-Length
	  set length(n) {
	    this.set('Content-Length', n);
	  },
	
	  // 以数字形式获取响应的内容长度Content-Length
	  get length() {
	    const len = this.header['content-length'];
	    const body = this.body;
	
	    if (null == len) {
	      if (!body) return;
	      if ('string' == typeof body) return Buffer.byteLength(body);
	      if (Buffer.isBuffer(body)) return body.length;
	      if (isJSON(body)) return Buffer.byteLength(JSON.stringify(body));
	      return;
	    }
	
	    return ~~len;
	  },
	
	  // 响应头是否经socket形式发出，用于在发生错误时检查客户端是否被通知
	  get headerSent() {
	    return this.res.headersSent;
	  },
	
	  // 等同this.append("Vary",field)，向响应头的Vary属性添加field值
	  vary(field) {
	    vary(this.res, field);
	  },
	
	  // 重定向，响应头的Location属性设置为跳转url；若无状态码设置，状态码设为302
	  // 根据请求头的accept属性设置响应头的content-type属性及this.body响应内容
	  // this.redirect('back');
	  // this.redirect('back', '/index.html');
	  // this.redirect('/login');
	  // this.redirect('http://google.com');
	  redirect(url, alt) {
	    // location
	    if ('back' == url) url = this.ctx.get('Referrer') || alt || '/';
	    this.set('Location', url);
	
	    // 当前状态码不是300、301、302、303、305、307、308重定向状态码，this.status赋值为302
	    if (!statuses.redirect[this.status]) this.status = 302;
	
	    // this.ctx.accepts('html')判断客户端对响应内容的偏好
	    if (this.ctx.accepts('html')) {
	      url = escape(url);
	      this.type = 'text/html; charset=utf-8';
	      this.body = `Redirecting to <a href="${url}">${url}</a>.`;
	      return;
	    }
	
	    // text
	    this.type = 'text/plain; charset=utf-8';
	    this.body = `Redirecting to ${url}.`;
	  },
	
	  // 设置响应头的'Content-Disposition'属性，提示下载，filename可指定下载文件名
	  attachment(filename) {
	    if (filename) this.type = extname(filename);
	    this.set('Content-Disposition', contentDisposition(filename));
	  },
	
	  // 设置或移除响应头的'Content-Type'属性，this.type = '.html'形式
	  set type(type) {
	    type = getType(type) || false;
	    if (type) {
	      this.set('Content-Type', type);
	    } else {
	      this.remove('Content-Type');
	    }
	  },
	
	  // 以时间设置响应头的'Last-Modified'属性，socket长连接的时候有效
	  set lastModified(val) {
	    if ('string' == typeof val) val = new Date(val);
	    this.set('Last-Modified', val.toUTCString());
	  },
	
	  // 以日期对象形式获取响应头的'Last-Modified'属性
	  get lastModified() {
	    const date = this.get('last-modified');
	    if (date) return new Date(date);
	  },
	
	  // 设置响应头的'ETag'属性，双引号包裹，如this.response.etag = 'md5hashsum'
	  set etag(val) {
	    if (!/^(W\/)?"/.test(val)) val = `"${val}"`;
	    this.set('ETag', val);
	  },
	
	  // 获取响应头的'ETag'属性
	  get etag() {
	    return this.get('ETag');
	  },
	
	  // 获取响应头中'Content-Type'属性即MIME文件类型，不包含参数
	  get type() {
	    const type = this.get('Content-Type');
	    if (!type) return '';
	    return type.split(';')[0];
	  },
	
	  // 校验响应头中'Content-Type'属性即MIME文件类型是否匹配传参数组中的任意项；若匹配返回匹配值
	  is(types) {
	    const type = this.type;
	    if (!types) return type || false;
	    if (!Array.isArray(types)) types = [].slice.call(arguments);
	    return typeis(type, types);
	  },
	
	  // 获取响应头的field属性
	  get(field) {
	    return this.header[field.toLowerCase()] || '';
	  },
	
	  // 双参数时，设置响应头的field属性为val，如this.set('Foo', ['bar', 'baz'])
	  // 单参数对象形式，根据参数对象属性设置响应头的对应属性，如this.set({ Accept: 'text/plain', 'X-API-Key': 'tobi' })
	  set(field, val) {
	    if (2 == arguments.length) {
	      if (Array.isArray(val)) val = val.map(String);
	      else val = String(val);
	      this.res.setHeader(field, val);
	    } else {
	      for (const key in field) {
	        this.set(key, field[key]);
	      }
	    }
	  },
	
	  // 响应头的field属性为数组，向数组中添加val，如this.append('Warning', '199 Miscellaneous warning')
	  append(field, val) {
	    const prev = this.get(field);
	
	    if (prev) {
	      val = Array.isArray(prev)
	        ? prev.concat(val)
	        : [prev].concat(val);
	    }
	
	    return this.set(field, val);
	  },
	
	  // 移除响应头的field属性
	  remove(field) {
	    this.res.removeHeader(field);
	  },
	
	  // 连接已终结，返回false；连接形式为socket，根据socket.writable判断响应是否可设置；连接为其他形式，返回真值
	  get writable() {
	    if (this.res.finished) return false;
	
	    const socket = this.res.socket;
	
	    if (!socket) return true;
	    return socket.writable;
	  },
	
	  // 获取状态码、状态码文案、响应头、响应{status,message,header,body}属性集合
	  inspect() {
	    if (!this.res) return;
	    const o = this.toJSON();
	    o.body = this.body;
	    return o;
	  },
	
	  // 获取状态码、状态码文案、响应头{status,message,header}属性集合
	  toJSON() {
	    return only(this, [
	      'status',
	      'message',
	      'header'
	    ]);
	  },
	
	  // 刷新响应头
	  flushHeaders() {
	    this.res.flushHeaders();
	  }
	};

### context.js

作为中间件的上下文，兼有处理请求、响应的能力，报错、把错误对象作为响应的能力。

	'use strict';
	
	// createError(status,msg,opts)构造错误对象{status,statusCode,message}
	const createError = require('http-errors');
	
	// httpAssert(value,status,msg,opts)当value为否值时，通过createError抛出错误{status,statusCode,message}
	const httpAssert = require('http-assert');
	
	// delegate(obj,propName).method(name)提供obj[name]代理调用obj.propName[name]方法
	const delegate = require('delegates');
	
	// http状态码类型
	const statuses = require('statuses');
	
	const proto = module.exports = {
	
	  inspect() {
	    return this.toJSON();
	  },
	
	  // 获取koa封装的请求、响应、应用对象信息，原始请求地址，node封装的请求、响应、socket对象信息
	  // {request,response,app,originalUrl,req,res,socket}
	  toJSON() {
	    return {
	      request: this.request.toJSON(),
	      response: this.response.toJSON(),
	      app: this.app.toJSON(),
	      originalUrl: this.originalUrl,
	      req: '<original node req>',
	      res: '<original node res>',
	      socket: '<original node socket>'
	    };
	  },
	
	  // this.assert(value,status,msg,opts)当value为否值时，抛出错误{status,statusCode,message}
	  assert: httpAssert,
	
	  // this.throw(status,msg,opts)构造错误对象{status,statusCode,message}
	  throw() {
	    throw createError.apply(null, arguments);
	  },
	
	  // 默认的错误处理函数，try-catch语句捕获错误时再调用this.onerror方法处理，同时触发app的error事件
	  // 错误对象err的expose为否，通过app.onError打印错误；为真，以err.message作为响应
	  onerror(err) {
	    if (null == err) return;
	
	    if (!(err instanceof Error)) err = new Error(`non-error thrown: ${err}`);
	
	    let headerSent = false;
	    if (this.headerSent || !this.writable) {// this.headerSent判断响应头是否被发送到客户端
	      headerSent = err.headerSent = true;
	    }
	
	    // 触发调用app的onError方法，err.status为404或err.expose为真或this.silent为真时，略过app.onError方法中的报错处理
	    // 否则通过console.error打印错误
	    this.app.emit('error', err, this);
	
	    if (headerSent) {
	      return;
	    }
	
	    const { res } = this;
	
	    // 重置响应头
	    if (typeof res.getHeaderNames === 'function') {
	      res.getHeaderNames().forEach(name => res.removeHeader(name));
	    } else {
	      res._headers = {}; // Node < 7.7
	    }
	
	    this.set(err.headers);
	
	    this.type = 'text';
	
	    // ENOENT support
	    if ('ENOENT' == err.code) err.status = 404;
	
	    if ('number' != typeof err.status || !statuses[err.status]) err.status = 500;
	
	    // err.expose为真，以err.message作为传输相应，否则以statuses[err.status]状态码文案作为响应
	    const code = statuses[err.status];
	    const msg = err.expose ? err.message : code;
	    this.status = err.status;
	    this.length = Buffer.byteLength(msg);
	    this.res.end(msg);
	  }
	};
	
	// delegate(proto, 'response')将this.response的方法、访问器、设置器赋值给proto关键字
	// Object.create(proto)形式构造中间件的参数ctx
	delegate(proto, 'response')
	  .method('attachment')// this.attachment(filename)设置响应头的'Content-Disposition'属性，指定下载文件名
	  .method('redirect')// this.redirect(back,url)重定向，未设置响应内容或状态码，状态码自动设为302
	  .method('remove')// this.remove(field)移除响应头的field属性
	  .method('vary')// this.vary(field)添加响应头的响应头的'Vary'属性
	  .method('set')// this.set(field,val)设置响应头的field属性为val
	  .method('append')// this.append(field,val)响应头的field属性数组添加val值
	  .method('flushHeaders')// this.flushHeaders()刷新响应头
	  .access('status')// 获取或设置状态码
	  .access('message')// 获取或设置状态码文案
	  .access('body')// 获取或设置响应内容
	  .access('length')// 获取或设置响应内容的长度
	  .access('type')// 获取或设置响应头中'Content-Type'属性即MIME文件类型，不包含参数
	  .access('lastModified')// 日期对象形式获取或设置响应头的'Last-Modified'属性，以判断请求是否被更新
	  .access('etag')// 获取或设置响应头的'ETag'属性，以判断请求是否被更新
	  .getter('headerSent')// 判断响应头是否经socket形式发出，用于在发生错误时检查客户端是否被通知
	  .getter('writable');// 判断是否可以设置响应
	
	delegate(proto, 'request')
	  .method('acceptsLanguages')// this.acceptsLanguages(languages)校验请求头中'accept-language'属性是否匹配传参数组中的任意项
	  .method('acceptsEncodings')// this.acceptsEncodings(encodings)校验请求头中'accept-encoding'属性是否匹配传参数组中的任意项
	  .method('acceptsCharsets')// this.acceptsCharsets(charsets)校验请求头中'accept-charset'属性是否匹配传参数组中的任意项
	  .method('accepts')// this.accepts(type)校验请求头中'accept'属性即客户端偏好响应类型是否匹配传参数组中的任意项；若匹配返回匹配值
	  .method('get')// this.get(field)获取请求头的field属性
	  .method('is')// this.is(types)校验请求头中'Content-Type'属性即MIME文件类型是否匹配传参数组中的任意项；若匹配返回匹配值
	  .access('querystring')// 以字符串形式获取或设置参数字符串
	  .access('idempotent')// 校验请求方法是否为'GET','HEAD','PUT','DELETE','OPTIONS','TRACE'中的一种
	  .access('socket')// 获取或设置请求的socket
	  .access('search')// 以字符串形式获取或设置参数字符串，带"?"
	  .access('method')// 获取或设置请求方法
	  .access('query')// 以对象形式获取或设置参数字符串
	  .access('path')// 获取或设置请求路径
	  .access('url')// 获取或设置请求的完整路径
	  .getter('origin')// 获取请求的协议名和域名
	  .getter('href')// 获取原始请求地址的全名
	  .getter('subdomains')// 获取请求的子域名
	  .getter('protocol')// 获取协议名http或https
	  .getter('host')// 获取域名，含端口
	  .getter('hostname')// 获取域名，不含端口
	  .getter('header')// 获取请求头
	  .getter('headers')// 获取请求头
	  .getter('secure')// 是否安全性更高的https协议
	  .getter('stale')// 判断请求是否未更新
	  .getter('fresh')// 判断请求是否被更新
	  .getter('ips')// 以数组形式获取请求的ip地址；http代理或负载均衡技术实现代理时，包含代理地址
	  .getter('ip');// 获取请求的ip地址
