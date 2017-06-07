# node-fetch 1.6.3

## 概述

fetch(url, options)，node端使用http、https模块发送请求用。

## 使用

    // plain text or html
    
    fetch('https://github.com/')
        .then(function(res) {
            return res.text();
        }).then(function(body) {
            console.log(body);
        });

    // json
    
    fetch('https://api.github.com/users/github')
        .then(function(res) {
            return res.json();
        }).then(function(json) {
            console.log(json);
        });

    // catching network error
    
    fetch('http://domain.invalid/')
        .catch(function(err) {
            console.log(err);
        });

    // stream
    
    fetch('https://assets-cdn.github.com/images/modules/logos_page/Octocat.png')
        .then(function(res) {
            var dest = fs.createWriteStream('./octocat.png');
            res.body.pipe(dest);
        });

    // buffer
    
    var fileType = require('file-type');
    fetch('https://assets-cdn.github.com/images/modules/logos_page/Octocat.png')
        .then(function(res) {
            return res.buffer();
        }).then(function(buffer) {
            fileType(buffer);
        });

    // meta
    
    fetch('https://github.com/')
        .then(function(res) {
            console.log(res.ok);
            console.log(res.status);
            console.log(res.statusText);
            console.log(res.headers.raw());
            console.log(res.headers.get('content-type'));
        });

    // post
    
    fetch('http://httpbin.org/post', { method: 'POST', body: 'a=1' })
        .then(function(res) {
            return res.json();
        }).then(function(json) {
            console.log(json);
        });

    // post with stream from resumer
    
    var resumer = require('resumer');
    var stream = resumer().queue('a=1').end();
    fetch('http://httpbin.org/post', { method: 'POST', body: stream })
        .then(function(res) {
            return res.json();
        }).then(function(json) {
            console.log(json);
        });

    // post with form-data (detect multipart)
    
    var FormData = require('form-data');
    var form = new FormData();
    form.append('a', 1);
    fetch('http://httpbin.org/post', { method: 'POST', body: form })
        .then(function(res) {
            return res.json();
        }).then(function(json) {
            console.log(json);
        });

    // post with form-data (custom headers)
    
    var FormData = require('form-data');
    var form = new FormData();
    form.append('a', 1);
    fetch('http://httpbin.org/post', { method: 'POST', body: form, headers: form.getHeaders() })
        .then(function(res) {
            return res.json();
        }).then(function(json) {
            console.log(json);
        });

    // node 0.12+, yield with co
    
    var co = require('co');
    co(function *() {
        var res = yield fetch('https://api.github.com/users/github');
        var json = yield res.json();
        console.log(res);
    });

## 源码

### index.js

    // url.parse(urlString[, parseQueryString[, slashesDenoteHost]])
    // 参数urlString为url字符串，参数parseQueryString为真时解析url参数
    // 参数slashesDenoteHost为真时将"//"、"/"中的内容解析为host属性
    var parse_url = require('url').parse;
    
    // url.resolve(from, to)拼接from、to，构成新的url字符串
    // url.resolve('/one/two/three', 'four');         // '/one/two/four'
    // url.resolve('http://example.com/', '/one');    // 'http://example.com/one'
    // url.resolve('http://example.com/one', '/two'); // 'http://example.com/two'
    var resolve_url = require('url').resolve;
    
    // https | http.request(options[, callback])发送请求
    // options.protocol，使用的协议，"http:"或"https:"
    // options.host，请求发送至的服务器的域名或IP地址，默认为"localhost"
    // options.hostname，options.host的别名; 为了支持url.parse()，hostname优于host
    // options.family，当解析host和hostname时使用的IP地址族; 有效值是4或6，当未指定时，则同时使用IP v4和v6
    // options.port，远程服务器的端口，默认为80
    // options.localAddress，为网络连接绑定的本地接口
    // options.socketPath，Unix域Socket（使用host:port或socketPath）
    // options.method，指定HTTP请求方法的字符串，默认为'GET'。
    // options.path，请求路径，默认为'/'
    // options.headers，包含请求头的对象
    // options.auth，基本身份验证，如'user:password'用来计算Authorization请求头
    // options.agent，控制Agent的行为，默认为undefined(对该主机和端口使用 http.globalAgent)
    //      也可以是Agent对象(显式地使用传入的Agent)、或false(创建一个新的使用默认值的Agent)
    // options.createConnection，函数，当不使用agent选项时，为请求创建一个socket或流
    // options.timeout，指定socket超时的毫秒数。 它设置了 socket 等待连接的超时时间
    // 参数callback会作为单次监听器被添加到'response'事件
    // http.request() 返回一个http.ClientRequest类的实例，http.ClientRequest为可写流
    var http = require('http');
    var https = require('https');
    
    var zlib = require('zlib');
    var stream = require('stream');
    
    var Body = require('./lib/body');
    var Response = require('./lib/response');
    var Headers = require('./lib/headers');
    var Request = require('./lib/request');
    var FetchError = require('./lib/fetch-error');
    
    module.exports = Fetch;
    module.exports.default = module.exports;
    
    /**
     * @param   Mixed    url   url字符串或Request实例
     * @param   Object   opts  { method, redirect, headers, follow, compress, counter, agent, timeout, size }
     * @return  Promise  发送请求并设置promise成功失败回调的执行机制，及响应的数据内容
     */
    function Fetch(url, opts) {
        if (!(this instanceof Fetch)) return new Fetch(url, opts);
    
        if (!Fetch.Promise) {
            throw new Error('native promise missing, set Fetch.Promise to your favorite alternative');
        }
    
        Body.Promise = Fetch.Promise;
    
        var self = this;
    
        return new Fetch.Promise(function(resolve, reject) {
            var options = new Request(url, opts);
    
            if (!options.protocol || !options.hostname) {
                throw new Error('only absolute urls are supported');
            }
    
            if (options.protocol !== 'http:' && options.protocol !== 'https:') {
                throw new Error('only http(s) protocols are supported');
            }
    
            var send;
            if (options.protocol === 'https:') {
                send = https.request;
            } else {
                send = http.request;
            }
    
            var headers = new Headers(options.headers);
    
            if (options.compress) {
                headers.set('accept-encoding', 'gzip,deflate');
            }
    
            if (!headers.has('user-agent')) {
                headers.set('user-agent', 'node-fetch/1.0 (+https://github.com/bitinn/node-fetch)');
            }
    
            if (!headers.has('connection') && !options.agent) {
                headers.set('connection', 'close');
            }
    
            if (!headers.has('accept')) {
                headers.set('accept', '*/*');
            }
    
            // detect form data input from form-data module, this hack avoid the need to pass multipart header manually
            if (!headers.has('content-type') && options.body && typeof options.body.getBoundary === 'function') {
                headers.set('content-type', 'multipart/form-data; boundary=' + options.body.getBoundary());
            }
    
            if (!headers.has('content-length') && /post|put|patch|delete/i.test(options.method)) {
                if (typeof options.body === 'string') {
                    headers.set('content-length', Buffer.byteLength(options.body));
                
                } else if (options.body && typeof options.body.getLengthSync === 'function') {
                    // for form-data 1.x
                    if (options.body._lengthRetrievers && options.body._lengthRetrievers.length == 0) {
                        headers.set('content-length', options.body.getLengthSync().toString());
                    // for form-data 2.x
                    } else if (options.body.hasKnownLength && options.body.hasKnownLength()) {
                        headers.set('content-length', options.body.getLengthSync().toString());
                    }
    
                } else if (options.body === undefined || options.body === null) {
                    headers.set('content-length', '0');
                }
            }
    
            options.headers = headers.raw();
    
            if (options.headers.host) {
                options.headers.host = options.headers.host[0];
            }
    
            var req = send(options);
            var reqTimeout;
    
            if (options.timeout) {
                req.once('socket', function(socket) {
                    reqTimeout = setTimeout(function() {
                        req.abort();
                        reject(new FetchError('network timeout at: ' + options.url, 'request-timeout'));
                    }, options.timeout);
                });
            }
    
            req.on('error', function(err) {
                clearTimeout(reqTimeout);
                reject(new FetchError('request to ' + options.url + ' failed, reason: ' + err.message, 'system', err));
            });
    
            req.on('response', function(res) {
                clearTimeout(reqTimeout);
    
                if (self.isRedirect(res.statusCode) && options.redirect !== 'manual') {
                    if (options.redirect === 'error') {
                        reject(new FetchError('redirect mode is set to error: ' + options.url, 'no-redirect'));
                        return;
                    }
    
                    if (options.counter >= options.follow) {
                        reject(new FetchError('maximum redirect reached at: ' + options.url, 'max-redirect'));
                        return;
                    }
    
                    if (!res.headers.location) {
                        reject(new FetchError('redirect location header missing at: ' + options.url, 'invalid-redirect'));
                        return;
                    }
    
                    // POST请求返回301/302响应，或纯303响应，根据响应的res.headers.location使用"get"方式发送请求
                    if (res.statusCode === 303
                        || ((res.statusCode === 301 || res.statusCode === 302) && options.method === 'POST'))
                    {
                        options.method = 'GET';
                        delete options.body;
                        delete options.headers['content-length'];
                    }
    
                    options.counter++;
    
                    resolve(Fetch(resolve_url(options.url, res.headers.location), options));
                    return;
                }
    
                // options.redirect="manual"时，由回调处理响应res
                var headers = new Headers(res.headers);
                if (options.redirect === 'manual' && headers.has('location')) {
                    headers.set('location', resolve_url(options.url, headers.get('location')));
                }
    
                // 转化响应
                var body = res.pipe(new stream.PassThrough());
                var response_options = {
                    url: options.url
                    , status: res.statusCode
                    , statusText: res.statusMessage
                    , headers: headers
                    , size: options.size
                    , timeout: options.timeout
                };
    
                var output;
    
                if (!options.compress || options.method === 'HEAD' || !headers.has('content-encoding') || res.statusCode === 204 || res.statusCode === 304) {
                    output = new Response(body, response_options);
                    resolve(output);
                    return;
                }
    
                var name = headers.get('content-encoding');
    
                // for gzip
                if (name == 'gzip' || name == 'x-gzip') {
                    body = body.pipe(zlib.createGunzip());
                    output = new Response(body, response_options);
                    resolve(output);
                    return;
    
                // for deflate
                } else if (name == 'deflate' || name == 'x-deflate') {
                    var raw = res.pipe(new stream.PassThrough());
                    raw.once('data', function(chunk) {
                        if ((chunk[0] & 0x0F) === 0x08) {
                            body = body.pipe(zlib.createInflate());
                        } else {
                            body = body.pipe(zlib.createInflateRaw());
                        }
                        output = new Response(body, response_options);
                        resolve(output);
                    });
                    return;
                }
    
                output = new Response(body, response_options);
                resolve(output);
                return;
            });
    
            if (typeof options.body === 'string') {
                req.write(options.body);
                req.end();
            } else if (options.body instanceof Buffer) {
                req.write(options.body);
                req.end()
            } else if (typeof options.body === 'object' && options.body.pipe) {
                options.body.pipe(req);
            } else if (typeof options.body === 'object') {
                req.write(options.body.toString());
                req.end();
            } else {
                req.end();
            }
        });
    
    };
    
    Fetch.prototype.isRedirect = function(code) {
        return code === 301 || code === 302 || code === 303 || code === 307 || code === 308;
    }
    
    Fetch.Promise = global.Promise;
    Fetch.Response = Response;
    Fetch.Headers = Headers;
    Fetch.Request = Request;

### lib/request.js

    var parse_url = require('url').parse;
    var Headers = require('./headers');
    var Body = require('./body');
    
    module.exports = Request;
    
    /**
     * @param   Mixed   input  Url or Request instance
     * @param   Object  init   { method, redirect, headers, follow, compress, counter, agent, timeout, size }
     */
    function Request(input, init) {
        var url, url_parsed;
    
        if (!(input instanceof Request)) {
            url = input;
            url_parsed = parse_url(url);
            input = {};
        } else {
            url = input.url;
            url_parsed = parse_url(url);
        }
    
        init = init || {};
    
        this.method = init.method || input.method || 'GET';
        this.redirect = init.redirect || input.redirect || 'follow';
        this.headers = new Headers(init.headers || input.headers || {});
        this.url = url;
    
        this.follow = init.follow !== undefined ?
            init.follow : input.follow !== undefined ?
            input.follow : 20;
        this.compress = init.compress !== undefined ?
            init.compress : input.compress !== undefined ?
            input.compress : true;
        this.counter = init.counter || input.counter || 0;
        this.agent = init.agent || input.agent;
    
        Body.call(this, init.body || this._clone(input), {
            timeout: init.timeout || input.timeout || 0,// 设置超时时间
            size: init.size || input.size || 0// 限制流的最大长度
        });
    
        this.protocol = url_parsed.protocol;
        this.hostname = url_parsed.hostname;
        this.port = url_parsed.port;
        this.path = url_parsed.path;
        this.auth = url_parsed.auth;
    }
    
    Request.prototype = Object.create(Body.prototype);
    
    Request.prototype.clone = function() {
        return new Request(this);
    };

### lib/response.js

    var http = require('http');
    var Headers = require('./headers');
    var Body = require('./body');
    
    module.exports = Response;
    
    function Response(body, opts) {
        opts = opts || {};
    
        this.url = opts.url;
        this.status = opts.status || 200;
        this.statusText = opts.statusText || http.STATUS_CODES[this.status];
        this.headers = new Headers(opts.headers);
        this.ok = this.status >= 200 && this.status < 300;
    
        // 通过Body构造函数添加json、text方法等
        Body.call(this, body, opts);
    }
    
    Response.prototype = Object.create(Body.prototype);
    
    Response.prototype.clone = function() {
        return new Response(this._clone(this), {
            url: this.url
            , status: this.status
            , statusText: this.statusText
            , headers: this.headers
            , ok: this.ok
        });
    };

### lib/headers.js

    module.exports = Headers;
    
    // 处理消息头
    function Headers(headers) {
    
        var self = this;
        this._headers = {};
    
        if (headers instanceof Headers) {
            headers = headers.raw();
        }
    
        for (var prop in headers) {
            if (!headers.hasOwnProperty(prop)) {
                continue;
            }
    
            if (typeof headers[prop] === 'string') {
                this.set(prop, headers[prop]);
    
            } else if (typeof headers[prop] === 'number' && !isNaN(headers[prop])) {
                this.set(prop, headers[prop].toString());
    
            } else if (headers[prop] instanceof Array) {
                headers[prop].forEach(function(item) {
                    self.append(prop, item.toString());
                });
            }
        }
    
    }
    
    Headers.prototype.get = function(name) {
        var list = this._headers[name.toLowerCase()];
        return list ? list[0] : null;
    };
    
    Headers.prototype.getAll = function(name) {
        if (!this.has(name)) {
            return [];
        }
    
        return this._headers[name.toLowerCase()];
    };
    
    Headers.prototype.forEach = function(callback, thisArg) {
        Object.getOwnPropertyNames(this._headers).forEach(function(name) {
            this._headers[name].forEach(function(value) {
                callback.call(thisArg, value, name, this)
            }, this)
        }, this)
    }
    
    Headers.prototype.set = function(name, value) {
        this._headers[name.toLowerCase()] = [value];
    };
    
    Headers.prototype.append = function(name, value) {
        if (!this.has(name)) {
            this.set(name, value);
            return;
        }
    
        this._headers[name.toLowerCase()].push(value);
    };
    
    Headers.prototype.has = function(name) {
        return this._headers.hasOwnProperty(name.toLowerCase());
    };
    
    Headers.prototype['delete'] = function(name) {
        delete this._headers[name.toLowerCase()];
    };
    
    Headers.prototype.raw = function() {
        return this._headers;
    };

### lib/body.js

    var convert = require('encoding').convert;
    var bodyStream = require('is-stream');
    var PassThrough = require('stream').PassThrough;
    var FetchError = require('./fetch-error');
    
    module.exports = Body;
    
    function Body(body, opts) {
    
        opts = opts || {};
    
        this.body = body;
        this.bodyUsed = false;
        this.size = opts.size || 0;
        this.timeout = opts.timeout || 0;
        this._raw = [];
        this._abort = false;
    }
    
    // 将响应作为json处理
    Body.prototype.json = function() {
        // for 204 No Content response, buffer will be empty, parsing it will throw error
        if (this.status === 204) {
            return Body.Promise.resolve({});
        }
    
        return this._decode().then(function(buffer) {
            return JSON.parse(buffer.toString());
        });
    };
    
    // 将响应作为text处理
    Body.prototype.text = function() {
        return this._decode().then(function(buffer) {
            return buffer.toString();
        });
    };
    
    // 将响应作为buffer处理
    Body.prototype.buffer = function() {
        return this._decode();
    };
    
    // 以promise形式处理响应this._decode().then(resolve,reject)，设定成功、失败回调的执行时机
    Body.prototype._decode = function() {
    
        var self = this;
    
        if (this.bodyUsed) {
            return Body.Promise.reject(new Error('body used already for: ' + this.url));
        }
    
        this.bodyUsed = true;
        this._bytes = 0;// 数据长度
        this._abort = false;// 超时中断标志
        this._raw = [];// 数据
    
        return new Body.Promise(function(resolve, reject) {
            var resTimeout;
    
            // body is string
            if (typeof self.body === 'string') {
                self._bytes = self.body.length;
                self._raw = [new Buffer(self.body)];
                return resolve(self._convert());
            }
    
            // body is buffer
            if (self.body instanceof Buffer) {
                self._bytes = self.body.length;
                self._raw = [self.body];
                return resolve(self._convert());
            }
    
            // allow timeout on slow response body
            if (self.timeout) {
                resTimeout = setTimeout(function() {
                    self._abort = true;
                    reject(new FetchError('response timeout at ' + self.url + ' over limit: ' + self.timeout, 'body-timeout'));
                }, self.timeout);
            }
    
            // handle stream error, such as incorrect content-encoding
            self.body.on('error', function(err) {
                reject(new FetchError('invalid response body at: ' + self.url + ' reason: ' + err.message, 'system', err));
            });
    
            // body is stream
            self.body.on('data', function(chunk) {
                if (self._abort || chunk === null) {
                    return;
                }
    
                if (self.size && self._bytes + chunk.length > self.size) {
                    self._abort = true;
                    reject(new FetchError('content size at ' + self.url + ' over limit: ' + 
                        self.size, 'max-size'));
                    return;
                }
    
                self._bytes += chunk.length;
                self._raw.push(chunk);
            });
    
            self.body.on('end', function() {
                if (self._abort) {
                    return;
                }
    
                clearTimeout(resTimeout);
                resolve(self._convert());
            });
        });
    
    };
    
    // 转化响应数据，通过this._decode方法将响应内容塞入this._raw中
    Body.prototype._convert = function(encoding) {
    
        encoding = encoding || 'utf-8';
    
        var ct = this.headers.get('content-type');
        var charset = 'utf-8';
        var res, str;
    
        // header
        if (ct) {
            // skip encoding detection altogether if not html/xml/plain text
            if (!/text\/html|text\/plain|\+xml|\/xml/i.test(ct)) {
                return Buffer.concat(this._raw);
            }
    
            res = /charset=([^;]*)/i.exec(ct);
        }
    
        // no charset in content type, peek at response body for at most 1024 bytes
        if (!res && this._raw.length > 0) {
            for (var i = 0; i < this._raw.length; i++) {
                str += this._raw[i].toString()
                if (str.length > 1024) {
                    break;
                }
            }
            str = str.substr(0, 1024);
        }
    
        // html5
        if (!res && str) {
            res = /<meta.+?charset=(['"])(.+?)\1/i.exec(str);
        }
    
        // html4
        if (!res && str) {
            res = /<meta[\s]+?http-equiv=(['"])content-type\1[\s]+?content=(['"])(.+?)\2/i.exec(str);
    
            if (res) {
                res = /charset=(.*)/i.exec(res.pop());
            }
        }
    
        // xml
        if (!res && str) {
            res = /<\?xml.+?encoding=(['"])(.+?)\1/i.exec(str);
        }
    
        // found charset
        if (res) {
            charset = res.pop();
    
            // prevent decode issues when sites use incorrect encoding
            // ref: https://hsivonen.fi/encoding-menu/
            if (charset === 'gb2312' || charset === 'gbk') {
                charset = 'gb18030';
            }
        }
    
        // turn raw buffers into a single utf-8 buffer
        return convert(
            Buffer.concat(this._raw)
            , encoding
            , charset
        );
    
    };
    
    Body.prototype._clone = function(instance) {
        var p1, p2;
        var body = instance.body;
    
        if (instance.bodyUsed) {
            throw new Error('cannot clone body after it is used');
        }
    
        // note: we can't clone the form-data object without having it as a dependency
        if (bodyStream(body) && typeof body.getBoundary !== 'function') {
            p1 = new PassThrough();
            p2 = new PassThrough();
            body.pipe(p1);
            body.pipe(p2);
            instance.body = p1;
            body = p2;
        }
    
        return body;
    }
    
    Body.Promise = global.Promise;

### lib/fetch-error.js

    module.exports = FetchError;
    
    // 构建错误对象
    function FetchError(message, type, systemError) {
    
        // hide custom error implementation details from end-users
        Error.captureStackTrace(this, this.constructor);
    
        this.name = this.constructor.name;
        this.message = message;
        this.type = type;
    
        if (systemError) {
            this.code = this.errno = systemError.code;
        }
    
    }
    
    require('util').inherits(FetchError, Error);