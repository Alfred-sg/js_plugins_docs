# koa-router 5.4.0

## apis

### Router

* let router = require('koa-router')(opts)，创建Router实例，用于管理路由中间件。
* router.[ "get" | "push" | "put" | "del" | ... ](name,path,...middlewares)，以标识符name注册匹配path路径的多个控制器middleware。
* router.use([ path | paths ],...middlewares)，为指定路径path或多个指定路径或任意路径注册多个控制器middleware。特别，当路径不是paths数组形式时，middleware可以是router.routes()方法获得的中间件，router.use将直接获取Layer实例，并完成中间件注册，即将Layer实例存入this.stack中。
* router.prefix(prefix)，设置url路径前缀，将改变每个已注册的Layer实例的路径。
* router.routes()，获取中间件，可以作为koa中间件完成挂载、或者调用router.use挂载为另一个Router实例的路由控制器；同时为koa上下文this注入matched、_matchedRoute、captures、params属性，可获取Router实例下所有匹配的注册路径、Router实例下注册的最匹配路径、查询参数数组、查询参数对象。
* router.allowedMethods(options)，当页面访问方式不是koa-router模块允许的请求方式或该路径下注册的请求方式不包含访问页面时的请求方式时，报错处理。
* router.all(name,path,middleware)，以标识符name为所有请求方式注册匹配path路径的多个控制器middleware。
* router.redirect(source,destination,code)，当页面访问source路径时，以状态码code重定向到destination路径。
* router.register(path,methods,middleware,opts)，为path路径、请求方式methods、控制器middleware，创建Layer实例，该Layer实例中添加参数中间件，并将Layer实例存入this.stack中。
* router.route(name)，获取标识符为name的Layer实例，即Layer实例的name属性为name。
* router.url(name,params)，获取标识符为name的Layer实例，以params查询参数拼接url路径输出。
* router.match(path,method)，返回{path,pathAndMethod}对象。其中，path属性获取匹配路径path的Layer实例，pathAndMethod属性获取匹配路径path及指定请求方式method的Layer实例。
* router.param(param,middleware)，为this.stack下各Layer实例添加参数中间件。
* router.url(path,params)，通过path基础路径和查询参数params设置url路径。

### Layer

* let layer = new Layer(path,methods,middleware,opts)，管理某路径下以请求方式methods注册的多个控制器middleware。
* layer.match(path)，判断路径path是否匹配Layer实例的路由规则。
* layer.params(path,captures,existingParams)，以existingParams对象形式或captures数组形式设置url参数；参数captures需要this.paramNames已注入值如[{name:""}]。
* layer.captures()，数组形式输出url参数值。
* layer.url(params) | layer.url.call({path},params)，基于this.path，将params数组或对象，转化成实际的url路径输出。
* layer.param(param,middleware)，以特定顺序注册参数中间件。
* layer.setPrefix(prefix)，设置url路径前缀，清空查询参数。

## 使用

### 基本使用

    var app = require('koa')();
    var router = require('koa-router')();
    
    router.get('/', function *(next) {...});
    
    app
      .use(router.routes())
      .use(router.allowedMethods());

### 多个控制器

    router.get(
      '/users/:id',
      function *(next) {
        this.user = yield User.findOne(this.params.id);
        yield next;
      },
      function *(next) {
        console.log(this.user);
        // => { id: 17, name: "Alex" }
      }
    );

### 嵌套路由

    var forums = new Router();
    var posts = new Router();
    
    posts.get('/', function *(next) {...});
    posts.get('/:pid', function *(next) {...});
    forums.use('/forums/:fid/posts', posts.routes(), posts.allowedMethods());
    
    // responds to "/forums/123/posts" and "/forums/123/posts/123"
    app.use(forums.routes());

## 源码

### lib/router.js

    var debug = require('debug')('koa-router');
    var HttpError = require('http-errors');
    
    // http请求方式
    var methods = require('methods');
    
    // 解析匹配路由，设定查询参数机制，添加查询茶树相关中间件
    var Layer = require('./layer');
    
    module.exports = Router;
    
    // 注册路由中间件
    function Router(opts) {
      if (!(this instanceof Router)) {
        return new Router(opts);
      }
    
      this.opts = opts || {};
      this.methods = this.opts.methods || [
        'HEAD',
        'OPTIONS',
        'GET',
        'PUT',
        'PATCH',
        'POST',
        'DELETE'
      ];
    
      this.params = {};// 缓存各参数中间件
      this.stack = [];// 挂载Layer实例
    };
    
    // 以http请求方式添加原型方法，以特定路由和请求方式注册中间件，创建Layer实例并存入this.stack
    // middleware可以多个参数形式添加多个
    methods.forEach(function (method) {
      Router.prototype[method] = function (name, path, middleware) {
        var middleware;
    
        if (typeof path === 'string' || path instanceof RegExp) {
          middleware = Array.prototype.slice.call(arguments, 2);
        } else {
          middleware = Array.prototype.slice.call(arguments, 1);
          path = name;
          name = null;
        }
    
        // 创建Layer实例，Layer实例中添加参数中间件，并添加到this.stack中
        this.register(path, [method], middleware, {
          name: name// Layer实例标识符
        });
    
        return this;
      };
    });
    
    Router.prototype.del = Router.prototype['delete'];
    
    // use(paths,...middlewares) | use(path,...middlewares) | use(...middlewares) 注册中间件，创建Layer实例并存入this.stack
    // 前一种方式指定匹配url路径；后两种方式middleware可携带router属性指向Layer实例，将该实例存入this.stack
    //    特别，koa-router模块注册的路由调用routes方法将获得携带router属性的koa中间件，也可作为Router的参数中间件
    // 最后一种匹配url路径".*"，即不限制
    Router.prototype.use = function () {
      var router = this;
      var middleware = Array.prototype.slice.call(arguments);
      var path;
    
      if (Array.isArray(middleware[0]) && typeof middleware[0][0] === 'string') {
        middleware[0].forEach(function (p) {
          router.use.apply(router, [p].concat(middleware.slice(1)));
        });
    
        return this;
      }
    
      if (typeof middleware[0] === 'string') {
        path = middleware.shift();
      }
    
      // middleware中携带router属性指向Layer实例，将该实例存入this.stack
      middleware = middleware.filter(function (fn) {
        if (fn.router) {
          fn.router.stack.forEach(function (layer) {
            if (path) layer.setPrefix(path);
            if (router.opts.prefix) layer.setPrefix(router.opts.prefix);
            router.stack.push(layer);
          });
    
          if (router.params) {
            Object.keys(router.params).forEach(function (key) {
              fn.router.param(key, router.params[key]);
            });
          }
    
          return false;
        }
    
        return true;
      });
    
      if (middleware.length) {
        router.register(path || '(.*)', [], middleware, {
          end: false
        });
      }
    
      return this;
    };
    
    // 设置url路径前缀，将改变每个已注册的Layer实例的路径
    Router.prototype.prefix = function (prefix) {
      prefix = prefix.replace(/\/$/, '');
    
      this.opts.prefix = prefix;
    
      this.stack.forEach(function (route) {
        route.setPrefix(prefix);
      });
    
      return this;
    };
    
    // 生成挂载于koa的中间件
    Router.prototype.routes = Router.prototype.middleware = function () {
      var router = this;
    
      var dispatch = function *dispatch(next) {
        debug('%s %s', this.method, this.path);// this为koa应用的上下文
    
        var path = router.opts.routerPath || this.routerPath || this.path;
    
        // path属性获取匹配路径path的Layer实例，pathAndMethod属性获取指定method请求方式的Layer实例(且与path匹配)
        var matched = router.match(path, this.method);
        var layer, i, ii;
    
        if (this.matched) {// this.matched赋值为匹配的koa-router的路由规则
          this.matched.push.apply(this.matched, matched.path);
        } else {
          this.matched = matched.path;
        }
    
        if (matched.pathAndMethod.length) {// 获取激活的koa-router中间件
          i = matched.pathAndMethod.length;
          
          var mostSpecificPath = matched.pathAndMethod[matched.pathAndMethod.length - 1].path
          this._matchedRoute = mostSpecificPath
    
          while (matched.route && i--) {
            layer = matched.pathAndMethod[i];// 激活的Layer实例
            ii = layer.stack.length;// Layer实例下的中间件
            this.captures = layer.captures(path, this.captures);// 数组形式输出url参数值
            this.params = layer.params(path, this.captures, this.params);// 设置url参数
            debug('dispatch %s %s', layer.path, layer.regexp);
    
            while (ii--) {
              if (layer.stack[ii].constructor.name === 'GeneratorFunction') {
                // 执行中间件，更改next；下一个匹配路由中间件处理完成后，由改变后的next调用本次匹配路由中间件
                next = layer.stack[ii].call(this, next);
              } else {
                next = Promise.resolve(layer.stack[ii].call(this, next));
              }
            }
          }
        }
    
        if (typeof next.next === 'function') {
          yield *next;
        } else {
          yield next;
        }
      };
    
      dispatch.router = this;
    
      return dispatch;
    };
    
    // 请求方式不在this.methods中或不是koa-router注册对应路径的请求方式，报错
    Router.prototype.allowedMethods = function (options) {
      options = options || {};
      var implemented = this.methods;
    
      return function *allowedMethods(next) {
        yield *next;
    
        var allowed = {};
    
        if (!this.status || this.status === 404) {
          this.matched.forEach(function (route) {
            route.methods.forEach(function (method) {
              allowed[method] = method;
            });
          });
    
          var allowedArr = Object.keys(allowed);
    
          if (!~implemented.indexOf(this.method)) {
            if (options.throw) {
              var notImplementedThrowable;
              if (typeof options.notImplemented === 'function') {
                notImplementedThrowable = options.notImplemented(); // set whatever the user returns from their function
              } else {
                notImplementedThrowable = new HttpError.NotImplemented();
              }
              throw notImplementedThrowable;
            } else {
              this.status = 501;
              this.set('Allow', allowedArr);
            }
          } else if (allowedArr.length) {
            if (this.method === 'OPTIONS') {
              this.status = 204;
            } else if (!allowed[this.method]) {
              if (options.throw) {
                var notAllowedThrowable;
                if (typeof options.methodNotAllowed === 'function') {
                  notAllowedThrowable = options.methodNotAllowed(); // set whatever the user returns from their function
                } else {
                  notAllowedThrowable = new HttpError.MethodNotAllowed();
                }
                throw notAllowedThrowable;
              } else {
                this.status = 405;
              }
            }
            this.set('Allow', allowedArr);
          }
        }
      };
    };
    
    // 以标识符name为各请求方式注册中间件middleware
    Router.prototype.all = function (name, path, middleware) {
      var middleware;
    
      if (typeof path === 'string') {
        middleware = Array.prototype.slice.call(arguments, 2);
      } else {
        middleware = Array.prototype.slice.call(arguments, 1);
        path = name;
        name = null;
      }
    
      this.register(path, methods, middleware, {
        name: name
      });
    
      return this;
    };
    
    // 当以任意请求方式访问source路径时，重定向到destination
    Router.prototype.redirect = function (source, destination, code) {
      if (source[0] !== '/') {
        source = this.url(source);
      }
    
      if (destination[0] !== '/') {
        destination = this.url(destination);
      }
    
      return this.all(source, function *() {
        this.redirect(destination);// this为koa应用上下文
        this.status = code || 301;
      });
    };
    
    // 创建Layer实例，Layer实例中添加参数中间件，并添加到this.stack中
    Router.prototype.register = function (path, methods, middleware, opts) {
      opts = opts || {};
    
      var stack = this.stack;
    
      var route = new Layer(path, methods, middleware, {
        end: opts.end === false ? opts.end : true,
        name: opts.name,
        sensitive: opts.sensitive || this.opts.sensitive || false,
        strict: opts.strict || this.opts.strict || false,
        prefix: opts.prefix || this.opts.prefix || "",
      });
    
      if (this.opts.prefix) {
        route.setPrefix(this.opts.prefix);
      }
    
      // 为当前路由添加参数中间件
      Object.keys(this.params).forEach(function (param) {
        route.param(param, this.params[param]);
      }, this);
    
      // 将Layer实例route添加到this.stack中，等嵌套长度无参数的Layer实例靠后于有参数的实例
      if (methods.length || !stack.length) {
        var added = false;
    
        if (!route.paramNames.length) {
          var routeNestingLevel = route.path.toString().split('/').length;
    
          added = stack.some(function (m, i) {
            var mNestingLevel = m.path.toString().split('/').length;
            var isParamRoute = !!m.paramNames.length;
            if (routeNestingLevel === mNestingLevel && isParamRoute) {
              return stack.splice(i, 0, route);
            }
          });
        }
    
        if (!added) stack.push(route);
      } else {
        stack.some(function (m, i) {
          if (!m.methods.length && i === stack.length - 1) {
            return stack.push(route);
          } else if (m.methods.length) {
            if (stack[i - 1]) {
              return stack.splice(i, 0, route);
            } else {
              return stack.unshift(route);
            }
          }
        });
      }
    
      return route;
    };
    
    // 获取标识符为name的Layer实例
    Router.prototype.route = function (name) {
      var routes = this.stack;
    
      for (var len = routes.length, i=0; i<len; i++) {
        if (routes[i].name && routes[i].name === name) {
          return routes[i];
        }
      }
    
      return false;
    };
    
    // 获取标识符为name的Layer实例，以params查询参数拼接url路径输出
    Router.prototype.url = function (name, params) {
      var route = this.route(name);
        
      if (route) {
        var args = Array.prototype.slice.call(arguments, 1);
        return route.url.apply(route, args);
      }
    
      return new Error("No route found for name: " + name);
    };
    
    // path属性获取匹配路径path的Layer实例，pathAndMethod属性获取指定method请求方式的Layer实例(且与path匹配)
    Router.prototype.match = function (path, method) {
      var layers = this.stack;
      var layer;
      var matched = {
        path: [],
        pathAndMethod: [],
        route: false
      };
    
      for (var len = layers.length, i = 0; i < len; i++) {
        layer = layers[i];
    
        debug('test %s %s', layer.path, layer.regexp);
    
        if (layer.match(path)) {
          matched.path.push(layer);
    
          if (layer.methods.length === 0 || ~layer.methods.indexOf(method)) {
            matched.pathAndMethod.push(layer);
            if (layer.methods.length) matched.route = true;
          }
        }
      }
    
      return matched;
    };
    
    // 各Layer实例添加参数中间件
    Router.prototype.param = function (param, middleware) {
      this.params[param] = middleware;
      this.stack.forEach(function (route) {
        route.param(param, middleware);
      });
      return this;
    };
    
    // 通过path基础路径和查询参数params设置url路径
    Router.url = function (path, params) {
        return Layer.prototype.url.call({path: path}, params);
    };

### layer.js 

    var debug = require('debug')('koa-router');
    
    // pathToRegExp(path,params)，再调用exec方法获取匹配的路径和参数
    // 如pathToRegExp("/items/:id",[name:"id",prefix:"/"]).exec("/items/1")后，获得["/items/1","1"]
    // pathToRegexp.parse("/items/:id")以数组形式获取url正则规则["/items",{name:"id",prefix:'/'}]
    // pathToRegexp.compile("/items/:id")格式化函数，将对象转化为url，如{id:1}转化为"/items/1"
    var pathToRegExp = require('path-to-regexp');
    
    module.exports = Layer;
    
    /**
     * 解析匹配路由，设定查询参数机制，添加查询茶树相关中间件
     *
     * @param {String|RegExp} 字符串或正则设定路径
     * @param {Array} http请求方式
     * @param {Array} 中间件，即控制器
     * @param {Object=} opts
     * @param {String=} opts.name 路由名称，标识符
     * @param {String=} opts.sensitive case sensitive (default: false)
     * @param {String=} opts.strict require the trailing slash (default: false)
     * @returns {Layer} this.name 路由名称，标识符
     *                  this.methods http请求方式
     *                  this.paramNames:{} [{name:""}]，查询参数
     *                  this.stack=[fn] 中间件，尚未挂载
     *                  this.path 字符串或正则字符串形式设定的url
     *                  this.regrex 路径解析正则函数库
     */
    function Layer(path, methods, middleware, opts) {
      this.opts = opts || {};
      this.name = this.opts.name || null;
      this.methods = [];
      this.paramNames = [];
      this.stack = Array.isArray(middleware) ? middleware : [middleware];
    
      methods.forEach(function(method) {
        var l = this.methods.push(method.toUpperCase());
        if (this.methods[l-1] === 'GET') {
          this.methods.unshift('HEAD');
        }
      }, this);
    
      this.stack.forEach(function(fn) {
        var type = (typeof fn);
        if (type !== 'function') {
          throw new Error(
            methods.toString() + " `" + (this.opts.name || path) +"`: `middleware` "
            + "must be a function, not `" + type + "`"
          );
        }
      }, this);
    
      this.path = path;
      this.regexp = pathToRegExp(path, this.paramNames, this.opts);
    
      debug('defined route %s %s', this.methods, this.opts.prefix + this.path);
    };
    
    // 判断路径是否匹配当前的路由规则
    Layer.prototype.match = function (path) {
      return this.regexp.test(path);
    };
    
    // 以existingParams对象形式或captures数组形式设置url参数，captures须this.paramNames已注入值如[{name:""}]
    Layer.prototype.params = function (path, captures, existingParams) {
      var params = existingParams || {};
    
      for (var len = captures.length, i=0; i<len; i++) {
        if (this.paramNames[i]) {
          var c = captures[i];
          params[this.paramNames[i].name] = c ? safeDecodeURIComponent(c) : c;
        }
      }
    
      return params;
    };
    
    // 数组形式输出url参数值
    Layer.prototype.captures = function (path) {
      return path.match(this.regexp).slice(1);
    };
    
    // this.path以字符串设定url正则规则，接受params数组或对象，转化成实际的url路径输出
    // 如var route = new Layer(['GET'], '/users/:id', fn);
    // route.url({ id: 123 }); // => "/users/123"
    Layer.prototype.url = function (params) {
      var args = params;
      var url = this.path;
      var toPath = pathToRegExp.compile(url);
    
      if (typeof params != 'object') {
        args = Array.prototype.slice.call(arguments);
      }
    
      if (args instanceof Array) {
        var tokens = pathToRegExp.parse(url);
        var replace = {};
        for (var len = tokens.length, i=0, j=0; i<len; i++) {
          if (tokens[i].name) replace[tokens[i].name] = args[j++];
        }
        return toPath(replace);
      } else {
        return toPath(params);
      }
    };
    
    // 按查询参数位置添加中间件，并将查询参数的值传入中间件中
    Layer.prototype.param = function (param, fn) {
      var stack = this.stack;
      var params = this.paramNames;
    
      // 将查询参数中的param属性作为参数传入控制器
      var middleware = function *(next) {
        next = fn.call(this, this.params[param], next);
        if (typeof next.next === 'function') {
          yield *next;
        } else {
          yield Promise.resolve(next);
        }
      };
      middleware.param = param;// 标记中间件以param方法添加
    
      params.forEach(function (p, i) {
        var prev = params[i - 1];
    
        if (param === p.name) {
          // insert param middleware in order params appear in path
          if (prev) {
            if (!stack.some(function (m, i) {
              if (m.param === prev.name) {// prev查询参数添加过中间件，当前添加的中间件插入其后
                return stack.splice(i, 0, middleware);
              }
            })) {
              // prev查询参数未添加过中间件，当前添加的中间件顺序插入
              stack.some(function (m, i) {
                if (!m.param) {
                  return stack.splice(i, 0, middleware);
                }
              });
            }
          } else {
            stack.unshift(middleware);
          }
        }
      });
    
      return this;
    };
    
    // 设置url路径前缀，清空查询参数
    Layer.prototype.setPrefix = function (prefix) {
      if (this.path) {
        this.path = prefix + this.path;
        this.paramNames = [];
        this.regexp = pathToRegExp(this.path, this.paramNames, this.opts);
      }
    
      return this;
    };
    
    function safeDecodeURIComponent(text) {
      try {
        return decodeURIComponent(text);
      } catch (e) {
        return text;
      }
    }
