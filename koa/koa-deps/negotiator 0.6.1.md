# negotiator 0.6.1

## 概述和使用

negotiator模块用于获取请求头的accept-charset、accept-encoding、accept-language、accept-contentType属性，或者判断传参与上述属性的匹配程度，以设置响应内容的传输形式。

### accept-contentType

	negotiator = new Negotiator(request)
	
	// 若请求头的accept-contentType设置为'text/html, application/*;q=0.2, image/jpeg;q=0.8'，输出['text/html', 'image/jpeg', 'application/*']
	negotiator.mediaTypes()
	
	availableMediaTypes = ['text/html', 'text/plain', 'application/json']
	// 以请求头的accept-contentType优先级排序输出contentType数组，如['text/html', 'application/json']
	negotiator.mediaTypes(availableMediaTypes)
	// 将优先级排序后的数组首项输出，即'text/html'
	negotiator.mediaType(availableMediaTypes)
	
### accept-language

	negotiator = new Negotiator(request)
	
	// 若请求头的accept-language属性为'en;q=0.8, es, pt'，输出['es', 'pt', 'en']
	negotiator.languages()
	
	availableLanguages = ['en', 'es', 'fr']
	negotiator.languages(availableLanguages)// -> ['es', 'en']
	language = negotiator.language(availableLanguages)// -> 'es'
	
### accept-charset

	negotiator = new Negotiator(request)
	
	// 若请求头的accept-charset属性为'utf-8, iso-8859-1;q=0.8, utf-7;q=0.2'，输出['utf-8', 'iso-8859-1', 'utf-7']
	negotiator.charsets()
	
	availableCharsets = ['utf-8', 'iso-8859-1', 'iso-8859-5']
	negotiator.charsets(availableCharsets)// -> ['utf-8', 'iso-8859-1']
	negotiator.charset(availableCharsets)// -> 'utf-8'
	
### accept-encoding

	negotiator = new Negotiator(request)
	
	// 若请求头的accept-encoding属性为'gzip, compress;q=0.2, identity;q=0.5'，输出['gzip', 'identity', 'compress']
	negotiator.encodings()
	
	availableEncodings = ['identity', 'gzip']
	negotiator.encodings(availableEncodings)// -> ['gzip', 'identity']

	negotiator.encoding(availableEncodings)// -> 'gzip'	
	
## 源码

### index.js

	'use strict';
	
	var modules = Object.create(null);
	
	module.exports = Negotiator;
	module.exports.Negotiator = Negotiator;
	
	function Negotiator(request) {
	  if (!(this instanceof Negotiator)) {
	    return new Negotiator(request);
	  }
	
	  this.request = request;
	}
	
	// charset()以请求头的accept-charset属性获取响应内容的字符编码形式charset
	// charset([charset])将参数provided按请求头的accept-charset属性优先级排序；charsets返回数组，charset返回数组首项
	// charset值可设定为'utf-8'、'iso-8859-1'、'iso-8859-5'等
	Negotiator.prototype.charset = function charset(available) {
	  var set = this.charsets(available);
	  return set && set[0];
	};
	
	Negotiator.prototype.charsets = function charsets(available) {
	  var preferredCharsets = loadModule('charset').preferredCharsets;
	  return preferredCharsets(this.request.headers['accept-charset'], available);
	};
	
	// encoding()以请求头的accept-encoding属性获取响应内容的数据传输形式encoding
	// encoding([encoding])将参数provided按请求头的accept-encoding属性优先级排序
	// encoding值可设定为"chunked"拆散后成束传输、"identity"完整内容传输、"gzip"压缩后传输
	Negotiator.prototype.encoding = function encoding(available) {
	  var set = this.encodings(available);
	  return set && set[0];
	};
	
	Negotiator.prototype.encodings = function encodings(available) {
	  var preferredEncodings = loadModule('encoding').preferredEncodings;
	  return preferredEncodings(this.request.headers['accept-encoding'], available);
	};
	
	// language()以请求头的accept-language属性获取响应内容的语言language
	// language([language])将参数provided按请求头的accept-language属性优先级排序
	// language值可设定为"zh-cn"，"en"等
	Negotiator.prototype.language = function language(available) {
	  var set = this.languages(available);
	  return set && set[0];
	};
	
	Negotiator.prototype.languages = function languages(available) {
	  var preferredLanguages = loadModule('language').preferredLanguages;
	  return preferredLanguages(this.request.headers['accept-language'], available);
	};
	
	// mediaType()以请求头的accept-contentType属性获取响应内容的传输内容contentType
	// mediaType([contentType])将参数provided按请求头的accept-contentType属性优先级排序
	// contentType值可设定为'text/html'+...params, 'text/plain', 'application/json'等
	Negotiator.prototype.mediaType = function mediaType(available) {
	  var set = this.mediaTypes(available);
	  return set && set[0];
	};
	
	Negotiator.prototype.mediaTypes = function mediaTypes(available) {
	  var preferredMediaTypes = loadModule('mediaType').preferredMediaTypes;
	  return preferredMediaTypes(this.request.headers.accept, available);
	};
	
	Negotiator.prototype.preferredCharset = Negotiator.prototype.charset;
	Negotiator.prototype.preferredCharsets = Negotiator.prototype.charsets;
	Negotiator.prototype.preferredEncoding = Negotiator.prototype.encoding;
	Negotiator.prototype.preferredEncodings = Negotiator.prototype.encodings;
	Negotiator.prototype.preferredLanguage = Negotiator.prototype.language;
	Negotiator.prototype.preferredLanguages = Negotiator.prototype.languages;
	Negotiator.prototype.preferredMediaType = Negotiator.prototype.mediaType;
	Negotiator.prototype.preferredMediaTypes = Negotiator.prototype.mediaTypes;
	
	function loadModule(moduleName) {
	  var module = modules[moduleName];
	
	  if (module !== undefined) {
	    return module;
	  }
	
	  switch (moduleName) {
	    case 'charset':
	      module = require('./lib/charset');
	      break;
	    case 'encoding':
	      module = require('./lib/encoding');
	      break;
	    case 'language':
	      module = require('./lib/language');
	      break;
	    case 'mediaType':
	      module = require('./lib/mediaType');
	      break;
	    default:
	      throw new Error('Cannot find module \'' + moduleName + '\'');
	  }
	
	  modules[moduleName] = module;
	
	  return module;
	}

### charset.js

	'use strict';
	
	// preferredCharsets(accept)以请求头的accept-charset属性获取响应内容的字符编码设置charset
	// preferredCharsets(accept,[charset])将参数provided按请求头的accept-charset属性优先级排序
	// charset值可设定为'utf-8'、'iso-8859-1'、'iso-8859-5'等
	module.exports = preferredCharsets;
	module.exports.preferredCharsets = preferredCharsets;
	
	var simpleCharsetRegExp = /^\s*([^\s;]+)\s*(?:;(.*))?$/;
	
	// 将请求头的accept-charset属性如'utf-8;q=0.8,iso-8859-1,iso-8859-5'解析为数组形式[{charset,q,i}]
	function parseAcceptCharset(accept) {
	  var accepts = accept.split(',');
	
	  for (var i = 0, j = 0; i < accepts.length; i++) {
	    var charset = parseCharset(accepts[i].trim(), i);
	
	    if (charset) {
	      accepts[j++] = charset;
	    }
	  }
	
	  accepts.length = j;
	
	  return accepts;
	}
	
	// 将单个charset如'utf-8;q=0.8'解析为对象形式{charset:"utf-8",q:"0.8",i}；q属性指定相应字符编码的权重，默认为1
	function parseCharset(str, i) {
	  var match = simpleCharsetRegExp.exec(str);
	  if (!match) return null;
	
	  var charset = match[1];
	  var q = 1;
	  if (match[2]) {
	    var params = match[2].split(';')
	    for (var i = 0; i < params.length; i ++) {
	      var p = params[i].trim().split('=');
	      if (p[0] === 'q') {
	        q = parseFloat(p[1]);
	        break;
	      }
	    }
	  }
	
	  return {
	    charset: charset,
	    q: q,
	    i: i
	  };
	}
	
	// 取出accepted中匹配参数charset的字符编码设置，依次以o、q属性作为优先级判断；返回{charset,q,i,s}或null
	function getCharsetPriority(charset, accepted, index) {
	  var priority = {o: -1, q: 0, s: 0};
	
	  for (var i = 0; i < accepted.length; i++) {
	    var spec = specify(charset, accepted[i], index);
	
	    if (spec && (priority.s - spec.s || priority.q - spec.q || priority.o - spec.o) < 0) {
	      priority = spec;
	    }
	  }
	
	  return priority;
	}
	
	// 以参数charset判断spec={charset,q,i}，匹配的spec赋值为{charset,q,i,s=1}；否则返回null
	// 特别当spec.charset="*"时，返回{charset,q,i,s=0}
	function specify(charset, spec, index) {
	  var s = 0;
	  if(spec.charset.toLowerCase() === charset.toLowerCase()){
	    s |= 1;
	  } else if (spec.charset !== '*' ) {
	    return null
	  }
	
	  return {
	    i: index,
	    o: spec.i,
	    q: spec.q,
	    s: s
	  }
	}
	
	// preferredCharsets(accept)以请求头的accept-charset属性获取响应内容的字符编码设置charset
	// preferredCharsets(accept,[charset])将参数provided按请求头的accept-charset属性优先级排序
	function preferredCharsets(accept, provided) {
	  // 将请求头的accept-charset属性解析为数组形式[{charset,q,i}]
	  var accepts = parseAcceptCharset(accept === undefined ? '*' : accept || '');
	
	  // 以权重q设置字符编码，如'utf-8;q=0.8,iso-8859-1'取"iso-8859-1"作为输出响应的字符编码设置
	  if (!provided) {
	    return accepts
	      .filter(isQuality)
	      .sort(compareSpecs)
	      .map(getFullCharset);
	  }
	
	  // 以参数provided特定字符编码数组排序请求头accept字符编码设置[{charset,q,i,s}|null]
	  var priorities = provided.map(function getPriority(type, index) {
	    return getCharsetPriority(type, accepts, index);
	  });
	
	  // 以优先级排序参数provided特定字符编码，剔除请求头中不包含的字符编码，返回[charset|undefined]
	  return priorities.filter(isQuality).sort(compareSpecs).map(function getCharset(priority) {
	    return provided[priorities.indexOf(priority)];
	  });
	}
	
	// 比较，用于设定优先级
	function compareSpecs(a, b) {
	  return (b.q - a.q) || (b.s - a.s) || (a.o - b.o) || (a.i - b.i) || 0;
	}
	
	// 用于设定响应的优先级
	function getFullCharset(spec) {
	  return spec.charset;
	}
	
	function isQuality(spec) {
	  return spec.q > 0;
	}

### encoding.js

	'use strict';
	
	// preferredEncodings(accept)以请求头的accept-encoding属性获取响应内容的数据传输形式encoding
	// preferredEncodings(accept,[encoding])将参数provided按请求头的accept-encoding属性优先级排序
	// encoding值可设定为"chunked"拆散后成束传输、"identity"完整内容传输、"gzip"压缩后传输
	module.exports = preferredEncodings;
	module.exports.preferredEncodings = preferredEncodings;
	
	var simpleEncodingRegExp = /^\s*([^\s;]+)\s*(?:;(.*))?$/;
	
	function parseAcceptEncoding(accept) {
	  var accepts = accept.split(',');
	  var hasIdentity = false;
	  var minQuality = 1;
	
	  for (var i = 0, j = 0; i < accepts.length; i++) {
	    var encoding = parseEncoding(accepts[i].trim(), i);
	
	    if (encoding) {
	      accepts[j++] = encoding;
	      hasIdentity = hasIdentity || specify('identity', encoding);
	      minQuality = Math.min(minQuality, encoding.q || 1);
	    }
	  }
	
	  if (!hasIdentity) {
	    accepts[j++] = {
	      encoding: 'identity',
	      q: minQuality,
	      i: i
	    };
	  }
	
	  accepts.length = j;
	
	  return accepts;
	}
	
	function parseEncoding(str, i) {
	  var match = simpleEncodingRegExp.exec(str);
	  if (!match) return null;
	
	  var encoding = match[1];
	  var q = 1;
	  if (match[2]) {
	    var params = match[2].split(';');
	    for (var i = 0; i < params.length; i ++) {
	      var p = params[i].trim().split('=');
	      if (p[0] === 'q') {
	        q = parseFloat(p[1]);
	        break;
	      }
	    }
	  }
	
	  return {
	    encoding: encoding,
	    q: q,
	    i: i
	  };
	}
	
	function getEncodingPriority(encoding, accepted, index) {
	  var priority = {o: -1, q: 0, s: 0};
	
	  for (var i = 0; i < accepted.length; i++) {
	    var spec = specify(encoding, accepted[i], index);
	
	    if (spec && (priority.s - spec.s || priority.q - spec.q || priority.o - spec.o) < 0) {
	      priority = spec;
	    }
	  }
	
	  return priority;
	}
	
	function specify(encoding, spec, index) {
	  var s = 0;
	  if(spec.encoding.toLowerCase() === encoding.toLowerCase()){
	    s |= 1;
	  } else if (spec.encoding !== '*' ) {
	    return null
	  }
	
	  return {
	    i: index,
	    o: spec.i,
	    q: spec.q,
	    s: s
	  }
	};
	
	function preferredEncodings(accept, provided) {
	  var accepts = parseAcceptEncoding(accept || '');
	
	  if (!provided) {
	    return accepts
	      .filter(isQuality)
	      .sort(compareSpecs)
	      .map(getFullEncoding);
	  }
	
	  var priorities = provided.map(function getPriority(type, index) {
	    return getEncodingPriority(type, accepts, index);
	  });
	
	  return priorities.filter(isQuality).sort(compareSpecs).map(function getEncoding(priority) {
	    return provided[priorities.indexOf(priority)];
	  });
	}
	
	function compareSpecs(a, b) {
	  return (b.q - a.q) || (b.s - a.s) || (a.o - b.o) || (a.i - b.i) || 0;
	}
	
	function getFullEncoding(spec) {
	  return spec.encoding;
	}
	
	function isQuality(spec) {
	  return spec.q > 0;
	}

### language.js

	'use strict';
	
	module.exports = preferredLanguages;
	module.exports.preferredLanguages = preferredLanguages;
	
	var simpleLanguageRegExp = /^\s*([^\s\-;]+)(?:-([^\s;]+))?\s*(?:;(.*))?$/;
	
	// parseAcceptLanguage(accept)以请求头的accept-language属性获取响应内容的语言language
	// parseAcceptLanguage(accept,[language])将参数provided按请求头的accept-language属性优先级排序
	// language值可设定为"zh-cn"，"en"等
	function parseAcceptLanguage(accept) {
	  var accepts = accept.split(',');
	
	  for (var i = 0, j = 0; i < accepts.length; i++) {
	    var langauge = parseLanguage(accepts[i].trim(), i);
	
	    if (langauge) {
	      accepts[j++] = langauge;
	    }
	  }
	
	  accepts.length = j;
	
	  return accepts;
	}
	
	function parseLanguage(str, i) {
	  var match = simpleLanguageRegExp.exec(str);
	  if (!match) return null;
	
	  var prefix = match[1],
	      suffix = match[2],
	      full = prefix;
	
	  if (suffix) full += "-" + suffix;
	
	  var q = 1;
	  if (match[3]) {
	    var params = match[3].split(';')
	    for (var i = 0; i < params.length; i ++) {
	      var p = params[i].split('=');
	      if (p[0] === 'q') q = parseFloat(p[1]);
	    }
	  }
	
	  return {
	    prefix: prefix,
	    suffix: suffix,
	    q: q,
	    i: i,
	    full: full
	  };
	}
	
	function getLanguagePriority(language, accepted, index) {
	  var priority = {o: -1, q: 0, s: 0};
	
	  for (var i = 0; i < accepted.length; i++) {
	    var spec = specify(language, accepted[i], index);
	
	    if (spec && (priority.s - spec.s || priority.q - spec.q || priority.o - spec.o) < 0) {
	      priority = spec;
	    }
	  }
	
	  return priority;
	}
	
	function specify(language, spec, index) {
	  var p = parseLanguage(language)
	  if (!p) return null;
	  var s = 0;
	  if(spec.full.toLowerCase() === p.full.toLowerCase()){
	    s |= 4;
	  } else if (spec.prefix.toLowerCase() === p.full.toLowerCase()) {
	    s |= 2;
	  } else if (spec.full.toLowerCase() === p.prefix.toLowerCase()) {
	    s |= 1;
	  } else if (spec.full !== '*' ) {
	    return null
	  }
	
	  return {
	    i: index,
	    o: spec.i,
	    q: spec.q,
	    s: s
	  }
	};
	
	function preferredLanguages(accept, provided) {
	  var accepts = parseAcceptLanguage(accept === undefined ? '*' : accept || '');
	
	  if (!provided) {
	    return accepts
	      .filter(isQuality)
	      .sort(compareSpecs)
	      .map(getFullLanguage);
	  }
	
	  var priorities = provided.map(function getPriority(type, index) {
	    return getLanguagePriority(type, accepts, index);
	  });
	
	  return priorities.filter(isQuality).sort(compareSpecs).map(function getLanguage(priority) {
	    return provided[priorities.indexOf(priority)];
	  });
	}
	
	function compareSpecs(a, b) {
	  return (b.q - a.q) || (b.s - a.s) || (a.o - b.o) || (a.i - b.i) || 0;
	}
	
	function getFullLanguage(spec) {
	  return spec.full;
	}
	
	function isQuality(spec) {
	  return spec.q > 0;
	}

### mediaType.js

	'use strict';
	
	// preferredMediaTypes(accept)以请求头的accept-contentType属性获取响应内容的传输内容contentType
	// preferredMediaTypes(accept,[contentType])将参数provided按请求头的accept-contentType属性优先级排序
	// contentType值可设定为'text/html'+...params, 'text/plain', 'application/json'等
	module.exports = preferredMediaTypes;
	module.exports.preferredMediaTypes = preferredMediaTypes;
	
	var simpleMediaTypeRegExp = /^\s*([^\s\/;]+)\/([^;\s]+)\s*(?:;(.*))?$/;
	
	function parseAccept(accept) {
	  var accepts = splitMediaTypes(accept);
	
	  for (var i = 0, j = 0; i < accepts.length; i++) {
	    var mediaType = parseMediaType(accepts[i].trim(), i);
	
	    if (mediaType) {
	      accepts[j++] = mediaType;
	    }
	  }
	
	  accepts.length = j;
	
	  return accepts;
	}
	
	function parseMediaType(str, i) {
	  var match = simpleMediaTypeRegExp.exec(str);
	  if (!match) return null;
	
	  var params = Object.create(null);
	  var q = 1;
	  var subtype = match[2];
	  var type = match[1];
	
	  if (match[3]) {
	    var kvps = splitParameters(match[3]).map(splitKeyValuePair);
	
	    for (var j = 0; j < kvps.length; j++) {
	      var pair = kvps[j];
	      var key = pair[0].toLowerCase();
	      var val = pair[1];
	
	      var value = val && val[0] === '"' && val[val.length - 1] === '"'
	        ? val.substr(1, val.length - 2)
	        : val;
	
	      if (key === 'q') {
	        q = parseFloat(value);
	        break;
	      }
	
	      params[key] = value;
	    }
	  }
	
	  return {
	    type: type,
	    subtype: subtype,
	    params: params,
	    q: q,
	    i: i
	  };
	}
	
	function getMediaTypePriority(type, accepted, index) {
	  var priority = {o: -1, q: 0, s: 0};
	
	  for (var i = 0; i < accepted.length; i++) {
	    var spec = specify(type, accepted[i], index);
	
	    if (spec && (priority.s - spec.s || priority.q - spec.q || priority.o - spec.o) < 0) {
	      priority = spec;
	    }
	  }
	
	  return priority;
	}
	
	function specify(type, spec, index) {
	  var p = parseMediaType(type);
	  var s = 0;
	
	  if (!p) {
	    return null;
	  }
	
	  if(spec.type.toLowerCase() == p.type.toLowerCase()) {
	    s |= 4
	  } else if(spec.type != '*') {
	    return null;
	  }
	
	  if(spec.subtype.toLowerCase() == p.subtype.toLowerCase()) {
	    s |= 2
	  } else if(spec.subtype != '*') {
	    return null;
	  }
	
	  var keys = Object.keys(spec.params);
	  if (keys.length > 0) {
	    if (keys.every(function (k) {
	      return spec.params[k] == '*' || (spec.params[k] || '').toLowerCase() == (p.params[k] || '').toLowerCase();
	    })) {
	      s |= 1
	    } else {
	      return null
	    }
	  }
	
	  return {
	    i: index,
	    o: spec.i,
	    q: spec.q,
	    s: s,
	  }
	}
	
	function preferredMediaTypes(accept, provided) {
	  var accepts = parseAccept(accept === undefined ? '*/*' : accept || '');
	
	  if (!provided) {
	    return accepts
	      .filter(isQuality)
	      .sort(compareSpecs)
	      .map(getFullType);
	  }
	
	  var priorities = provided.map(function getPriority(type, index) {
	    return getMediaTypePriority(type, accepts, index);
	  });
	
	  return priorities.filter(isQuality).sort(compareSpecs).map(function getType(priority) {
	    return provided[priorities.indexOf(priority)];
	  });
	}
	
	function compareSpecs(a, b) {
	  return (b.q - a.q) || (b.s - a.s) || (a.o - b.o) || (a.i - b.i) || 0;
	}
	
	function getFullType(spec) {
	  return spec.type + '/' + spec.subtype;
	}
	
	function isQuality(spec) {
	  return spec.q > 0;
	}
	
	function quoteCount(string) {
	  var count = 0;
	  var index = 0;
	
	  while ((index = string.indexOf('"', index)) !== -1) {
	    count++;
	    index++;
	  }
	
	  return count;
	}
	
	function splitKeyValuePair(str) {
	  var index = str.indexOf('=');
	  var key;
	  var val;
	
	  if (index === -1) {
	    key = str;
	  } else {
	    key = str.substr(0, index);
	    val = str.substr(index + 1);
	  }
	
	  return [key, val];
	}
	
	function splitMediaTypes(accept) {
	  var accepts = accept.split(',');
	
	  for (var i = 1, j = 0; i < accepts.length; i++) {
	    if (quoteCount(accepts[j]) % 2 == 0) {
	      accepts[++j] = accepts[i];
	    } else {
	      accepts[j] += ',' + accepts[i];
	    }
	  }
	
	  accepts.length = j + 1;
	
	  return accepts;
	}
	
	function splitParameters(str) {
	  var parameters = str.split(';');
	
	  for (var i = 1, j = 0; i < parameters.length; i++) {
	    if (quoteCount(parameters[j]) % 2 == 0) {
	      parameters[++j] = parameters[i];
	    } else {
	      parameters[j] += ';' + parameters[i];
	    }
	  }
	
	  parameters.length = j + 1;
	
	  for (var i = 0; i < parameters.length; i++) {
	    parameters[i] = parameters[i].trim();
	  }
	
	  return parameters;
	}