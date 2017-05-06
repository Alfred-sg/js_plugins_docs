# mime-types 2.12.5

## 概述和使用

mime-types用于获取mime类型及其编码。

	var mime = require('mime-types')
	
	// mime.lookup通过文件名获取mime类型
	mime.lookup('json')             // 'application/json'
	mime.lookup('.md')              // 'text/x-markdown'
	mime.lookup('file.html')        // 'text/html'
	mime.lookup('folder/file.js')   // 'application/javascript'
	mime.lookup('folder/.htaccess') // false
	
	// 以待编码形式获取mime类型
	mime.contentType('markdown')  // 'text/x-markdown; charset=utf-8'
	mime.contentType('file.json') // 'application/json; charset=utf-8'
	mime.contentType(path.extname('/path/to/file.json')) // 'application/json; charset=utf-8'
	
	// 通过mime类型获取文件扩展名
	mime.extension('application/octet-stream') // 'bin'
	
	// 获取编码
	mime.charset('text/x-markdown') // 'UTF-8'
	
## 源码

	'use strict'
	
	var db = require('mime-db')// mime统计
	var extname = require('path').extname
	
	var extractTypeRegExp = /^\s*([^;\s]*)(?:;|\s|$)/
	var textTypeRegExp = /^text\//i
	
	exports.charset = charset// 获取mime类型type的编码格式
	exports.charsets = { lookup: charset }
	exports.contentType = contentType// 通过文件扩展名或mime类型获取mime类型，并添加编码字符串设定
	exports.extension = extension// 通过mime类型获取文件扩展名
	exports.extensions = Object.create(null)// mime类型到文件扩展名的映射
	exports.lookup = lookup// 通过文件扩展名或包含扩展名的文件名、路径获取mime类型
	exports.types = Object.create(null)// 文件扩展名到mime类型的映射
	
	// Populate the extensions/types maps
	populateMaps(exports.extensions, exports.types)
	
	// 获取mime类型type的编码格式，mime类型为"text/*text"默认设定为"utf-8"
	function charset (type) {
	  if (!type || typeof type !== 'string') {
	    return false
	  }
	
	  var match = extractTypeRegExp.exec(type)
	  var mime = match && db[match[1].toLowerCase()]
	
	  if (mime && mime.charset) {
	    return mime.charset
	  }
	
	  if (match && textTypeRegExp.test(match[1])) {
	    return 'UTF-8'
	  }
	
	  return false
	}
	
	// 获取mime类型传输内容类型，并设定编码格式
	function contentType (str) {
	  if (!str || typeof str !== 'string') {
	    return false
	  }
	
	  var mime = str.indexOf('/') === -1
	    ? exports.lookup(str)// 通过文件扩展名获取mime类型
	    : str
	
	  if (!mime) {
	    return false
	  }
	
	  // mime类型无设置编码格式，拼接charset=编码格式文本
	  if (mime.indexOf('charset') === -1) {
	    var charset = exports.charset(mime)
	    if (charset) mime += '; charset=' + charset.toLowerCase()
	  }
	
	  return mime
	}
	
	// 通过mime类型获取扩展名
	function extension (type) {
	  if (!type || typeof type !== 'string') {
	    return false
	  }
	
	  var match = extractTypeRegExp.exec(type)
	
	  var exts = match && exports.extensions[match[1].toLowerCase()]
	
	  if (!exts || !exts.length) {
	    return false
	  }
	
	  return exts[0]
	}
	
	// 通过文件扩展名或包含扩展名的文件名、路径获取mime类型
	function lookup (path) {
	  if (!path || typeof path !== 'string') {
	    return false
	  }
	
	  var extension = extname('x.' + path)
	    .toLowerCase()
	    .substr(1)
	
	  if (!extension) {
	    return false
	  }
	
	  return exports.types[extension] || false
	}
	
	// 为mime类型到文件扩展名的映射exports.extensions、文件扩展名到mime类型的映射exports.types注入内容
	function populateMaps (extensions, types) {
	  // source preference (least -> most)
	  var preference = ['nginx', 'apache', undefined, 'iana']
	
	  Object.keys(db).forEach(function forEachMimeType (type) {
	    var mime = db[type]
	    var exts = mime.extensions
	
	    if (!exts || !exts.length) {
	      return
	    }
	
	    // 存储{"application/javascript":["js"],"application/java-vm":["class"]}等，mime类型映射文件扩展名
	    extensions[type] = exts
	
	    for (var i = 0; i < exts.length; i++) {
	      var extension = exts[i]
	
	      // types[extension]已赋值，意味db[propName].extensions不同propName属性下的extensions数组有相同项
	      // exports.types扩展名到mime类型的映射以db[propName].source属性'nginx'优先级高于'apache'
	      if (types[extension]) {
	        var from = preference.indexOf(db[types[extension]].source)
	        var to = preference.indexOf(mime.source)
	
	        if (types[extension] !== 'application/octet-stream' &&
	          (from > to || (from === to && types[extension].substr(0, 12) === 'application/'))) {
	          // skip the remapping
	          continue
	        }
	      }
	
	      // 存储{"js":"application/javascript","class":"application/java-vm"}等，文件扩展名映射mime类型
	      types[extension] = type
	    }
	  })
	}

