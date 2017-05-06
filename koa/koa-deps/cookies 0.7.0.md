# cookies 0.7.0

## 概述和使用

cookies模块用于校验请求头中的cookie，识别用户身份；设置响应头中的set-cookie，发送给客户端，以作为客户端请求时发送给服务器的cookie。

	var http    = require( "http" )
	var Cookies = require( "cookies" )
	
	server = http.createServer( function( req, res ) {
		// 传参req、res用于获取请求消息头、设置响应消息头，keys设置加密密钥
	  var cookies = new Cookies( req, res, { "keys": keys } )
	    , unsigned, signed, tampered
	
	  if ( req.url == "/set" ) {
	    cookies
	      // 允许客户端通过document.cookie获取和设置cookie，unsigned="foo"
	      .set( "unsigned", "foo", { httpOnly: false } )
	
	      // 响应头发送signed="bar"的同时，发送密文signed.sig=require("crypto").createHamc("md5").update("bar").digest("base64")
	      .set( "signed", "bar", { signed: true } )
	
	      // 手动设置"tampered.sig"虚拟密文，模拟黑客篡改cookie，校验时将永不通过
	      .set( "tampered", "baz" )
	      .set( "tampered.sig", "bogus" )
	
	    res.writeHead( 302, { "Location": "/" } )
	    return res.end( "Now let's check." )
	  }
	
	  // 获取cookie的值为"foo"，可能经过客户端改写
	  unsigned = cookies.get( "unsigned" )
	  
	  // 因请求头cookie中，signed.sig的值匹配signed的值经keys加密后的数据，返回"bar"
	  signed = cookies.get( "signed", { signed: true } )
	  // 因请求头cookie中，tampered.sig不嗯给你匹配tampered经keys加密后的数据，返回undefined；表明身份识别未通过，可进行后续处理
	  tampered = cookies.get( "tampered", { signed: true } )
	
	  res.writeHead( 200, { "Content-Type": "text/plain" } )
	  res.end(
	    "unsigned expected: foo\n\n" +
	    "unsigned actual: " + unsigned + "\n\n" +
	    "signed expected: bar\n\n" +
	    "signed actual: " + signed + "\n\n" +
	    "tampered expected: undefined\n\n"+
	    "tampered: " + tampered + "\n\n"
	  )
	})

## 源码

	'use strict'
	
	var deprecate = require('depd')('cookies')
	
	// 基于crypto模块实现sign方法加密、verify比较密文、index比较密钥并获取密钥序号
	var Keygrip = require('keygrip')
	
	var http = require('http')
	var cache = {}
	
	var fieldContentRegExp = /^[\u0009\u0020-\u007e\u0080-\u00ff]+$/;
	
	var sameSiteRegExp = /^(?:lax|strict)$/i
	
	// 参数request请求，参数response响应
	// 参数options配置项{keys,secure}，其中keys作为keygrip加密算法的密钥串
	// 作为构造函数使用
	function Cookies(request, response, options) {
	  if (!(this instanceof Cookies)) return new Cookies(request, response, options)
	
	  this.secure = undefined
	  this.request = request
	  this.response = response
	
	  if (options) {
	    if (Array.isArray(options)) {
	      deprecate('"keys" argument; provide using options {"keys": [...]}')
	      this.keys = new Keygrip(options)
	    } else if (options.constructor && options.constructor.name === 'Keygrip') {
	      deprecate('"keys" argument; provide using options {"keys": keygrip}')
	      this.keys = options
	    } else {
	      this.keys = Array.isArray(options.keys) ? new Keygrip(options.keys) : options.keys
	      this.secure = options.secure
	    }
	  }
	}
	
	// 参数name待获取的cookie属性名
	// 参数opts={signed:true}是否以name+".signed"作为cookie属性名获取密文；this.keys有值时也获取密文
	// get(name) opts.signed为否且this.keys不存在时，以name作为cookie属性名取cookie的值
	// get(name,{signed:true}) 若请求头中的cookie[name+".sig"]匹配name经this.keys加密后的某项密文
	//    更新响应头的cookie有效期，返回cookie[name]值
	//    不匹配，无返回，响应头的cookie[name+".sig"]置为""，使cookie[name]校验永远不能通过
	Cookies.prototype.get = function(name, opts) {
	  var sigName = name + ".sig"
	    , header, match, value, remote, data, index
	    , signed = opts && opts.signed !== undefined ? opts.signed : !!this.keys
	
	  header = this.request.headers["cookie"]// 拼接字符串形式"key=value;"
	  if (!header) return
	
	  match = header.match(getPattern(name))
	  if (!match) return
	
	  value = match[1]
	  if (!opts || !signed) return value
	
	  remote = this.get(sigName)// 获取请求头cookie属性的name+".sig"后接的value值，即密文
	  if (!remote) return
	
	  data = name + "=" + value
	  if (!this.keys) throw new Error('.keys required for signed cookies');
	
	  // 数据data经this.keys加密后是否命中密文；若命中，获取密钥序号；不能命中，即密文遭客户端篡改
	  index = this.keys.index(data, remote)
	
	  // 密文遭客户端传给，无返回值，用于服务器校验客户端的身份，识别后作后续处理
	  // 设定响应头cookie属性sigName为""，随后的请求获取cookie名为name的cooike将永远返回undefined，即不匹配
	  if (index < 0) {
	    this.set(sigName, null, {path: "/", signed: false })
	
	  // 客户端身份识别通过，重设cookie的有效期后作为响应头内容输出
	  } else {
	    index && this.set(sigName, this.keys.sign(data), { signed: false })
	    return value
	  }
	};
	
	// 设置响应头的cookie，name=value，参数opts={secure}用于设置cooike的secure属性
	Cookies.prototype.set = function(name, value, opts) {
	  var res = this.response
	    , req = this.request
	    , headers = res.getHeader("Set-Cookie") || []
	    , secure = this.secure !== undefined ? !!this.secure : req.protocol === 'https' || req.connection.encrypted
	    , cookie = new Cookie(name, value, opts)
	    , signed = opts && opts.signed !== undefined ? opts.signed : !!this.keys
	
	  if (typeof headers == "string") headers = [headers]
	
	  if (!secure && opts && opts.secure) {
	    throw new Error('Cannot send secure cookie over unencrypted connection')
	  }
	
	  cookie.secure = secure
	  if (opts && "secure" in opts) cookie.secure = opts.secure
	
	  if (opts && "secureProxy" in opts) {
	    deprecate('"secureProxy" option; use "secure" option, provide "secure" to constructor if needed')
	    cookie.secure = opts.secureProxy
	  }
	
	  // pushCookie(cookies,cookie)，根据cookie替换或追加响应头的set-cookies属性文本
	  headers = pushCookie(headers, cookie)
	
	  if (opts && signed) {
	    if (!this.keys) throw new Error('.keys required for signed cookies');
	    cookie.value = this.keys.sign(cookie.toString())
	    cookie.name += ".sig"
	    headers = pushCookie(headers, cookie)
	  }
	
	  // 设置响应头的'Set-Cookie'属性
	  var setHeader = res.set ? http.OutgoingMessage.prototype.setHeader : res.setHeader
	  setHeader.call(res, 'Set-Cookie', headers)
	  return this
	};
	
	// 设置和管理响应头的某个cookie，toHeader转化为响应头文本
	function Cookie(name, value, attrs) {
	  if (!fieldContentRegExp.test(name)) {
	    throw new TypeError('argument name is invalid');
	  }
	
	  if (value && !fieldContentRegExp.test(value)) {
	    throw new TypeError('argument value is invalid');
	  }
	
	  value || (this.expires = new Date(0))
	
	  this.name = name
	  this.value = value || ""
	
	  for (var name in attrs) {
	    this[name] = attrs[name]
	  }
	
	  if (this.path && !fieldContentRegExp.test(this.path)) {
	    throw new TypeError('option path is invalid');
	  }
	
	  if (this.domain && !fieldContentRegExp.test(this.domain)) {
	    throw new TypeError('option domain is invalid');
	  }
	
	  if (this.sameSite && this.sameSite !== true && !sameSiteRegExp.test(this.sameSite)) {
	    throw new TypeError('option sameSite is invalid')
	  }
	}
	
	Cookie.prototype.path = "/";// 默认同域名下所有路径发起请求都携带cooike
	Cookie.prototype.expires = undefined;// 默认寿命为当前回话
	Cookie.prototype.domain = undefined;// 只允许当前网站域名发送cooike
	Cookie.prototype.httpOnly = true;// 不允许客户端解析设置cooike
	Cookie.prototype.sameSite = false;// cooike是否被当前域名独有，浏览器端不能将cookie交由其他域名发起请求
	Cookie.prototype.secure = false;// 只能使用ssl或https链接
	Cookie.prototype.overwrite = false;// cooike是否允许被重写，cookies插件内部使用
	
	// 以拼接字符串形式转化为响应头的set-cooike属性单个cookie的部分内容name+"="+value，遵循URL编码
	Cookie.prototype.toString = function() {
	  return this.name + "=" + this.value
	};
	
	// 以拼接字符串形式转化为响应头的set-cooike属性，多个cookie须拼接
	// 响应头的set-cookie属性设置 name=value [ ;expires=date][ ;domain=domain][ ;path=path][ ;secure]
	//    由服务器传给客户端，客户端将其作为请求头的cooike属性，只包含name=value，作为辨识用户的标识
	//    expires属性用于执行cooike的寿命；若超过约定时间，客户端将予以删除；默认寿命在一次回话中，浏览器关闭即删除
	//    domain属性用于指定cookie将被发送到哪些域名下，可设置为当前网站的子域名；默认为创建cookie的页面所在域名
	//    path属性用于指定哪些页面将发送cookie，正则匹配机制；如"/"设置意味"/about"请求路径将发送cookie
	//    samesite属性可设置为"strict"或"lax"；"strict"严格模式下其他网站发起的请求中不允许包含该cooike
	//        "lax"宽松模式可以，造成CSRF攻击的可能
	//    secure标记cookie将以ssl或https协议发送；https链接默认设置secure标记
	//    httponly标记设置cookie不能被前端的document.cookie获取并设置
	Cookie.prototype.toHeader = function() {
	  var header = this.toString()
	
	  if (this.maxAge) this.expires = new Date(Date.now() + this.maxAge);
	
	  if (this.path     ) header += "; path=" + this.path
	  if (this.expires  ) header += "; expires=" + this.expires.toUTCString()
	  if (this.domain   ) header += "; domain=" + this.domain
	  if (this.sameSite ) header += "; samesite=" + (this.sameSite === true ? 'strict' : this.sameSite.toLowerCase())
	  if (this.secure   ) header += "; secure"
	  if (this.httpOnly ) header += "; httponly"
	
	  return header
	};
	
	// 获取或设置cooike的寿命
	Object.defineProperty(Cookie.prototype, 'maxage', {
	  configurable: true,
	  enumerable: true,
	  get: function () { return this.maxAge },
	  set: function (val) { return this.maxAge = val }
	});
	deprecate.property(Cookie.prototype, 'maxage', '"maxage"; use "maxAge" instead')
	
	// 以name获取cookie字符串的正则表达式，返回值match[1]为cookie字符串name属性的值
	function getPattern(name) {
	  if (cache[name]) return cache[name]
	
	  return cache[name] = new RegExp(
	    "(?:^|;) *" +
	    name.replace(/[-[\]{}()*+?.,\\^$|#\s]/g, "\\$&") +
	    "=([^;]*)"
	  )
	}
	
	// 参数cookies响应头已有的set-cookies属性，参数cookie当前模块的cookie对象
	// 若cookie.overwrite为真，替换cookies中cookie的同名属性；为否，追加由cookie转化的响应头set-cookies属性文本
	function pushCookie(cookies, cookie) {
	  if (cookie.overwrite) {
	    cookies = cookies.filter(function(c) { return c.indexOf(cookie.name+'=') !== 0 })
	  }
	  cookies.push(cookie.toHeader())
	  return cookies
	}
	
	// express框架中使用，请求req、响应res挂载Cookies实例
	Cookies.connect = Cookies.express = function(keys) {
	  return function(req, res, next) {
	    req.cookies = res.cookies = new Cookies(req, res, {
	      keys: keys
	    })
	
	    next()
	  }
	}
	
	Cookies.Cookie = Cookie
	
	module.exports = Cookies

## 参考

* [HTTP cookies 详解](http://blog.csdn.net/lijing198997/article/details/9378047)
* [SameSite Cookie，防止 CSRF 攻击](http://www.tuicool.com/articles/3ayyMrZ)