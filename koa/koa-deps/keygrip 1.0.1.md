# keygrip 1.0.1

## 概述和使用

keygrip通过node内置模块crypto的createHmac加密数据、比较密文、比较密钥。

### 加密数据

	keylist = ["SEKRIT3", "SEKRIT2", "SEKRIT1"]
	keys = Keygrip(keylist)
	
	// 以密钥keylist[0]加密数据"bieberschnitzel"
	hash = keys.sign("bieberschnitzel")
	
### 密文比较

	keylist = ["SEKRIT3", "SEKRIT2", "SEKRIT1"]
	keys = Keygrip(keylist)
	
	// 数据"bieberschnitzel"经keylist密钥串顺序加密后，能否同密文hash等值，意味hash是数据"bieberschnitzel"经keylist密钥串中某一密钥加密后的值
	matched = keys.verify("bieberschnitzel", hash)
	
### 密钥比较

	keylist = ["SEKRIT3", "SEKRIT2", "SEKRIT1"]
	keys = Keygrip(keylist)
	
	// 数据"bieberschnitzel"若经keylist某一密钥加密后构成密文hash，返回那一密钥在keylist中的序号
	index = keys.index("bieberschnitzel", hash)
	
## 源码

	// node内置加密模块crypto，基于OpenSSL库实现的加密技术
	
	// 以算法"md5"、"sha1"、"sha256"、"sha512"、"ripemd160"生成hash对象，哈希运算消息认证码
	//    加密后不可逆，可通过彩虹表查找加密前的文本
	// "md5"全称Message-Digest Algorithm 5，信息-摘要算法；把一个任意长度的字节串变换成一定长的大整数
	// "sha1"全称是Secure Hash Algorithm，安全哈希算法
	// const hash = crypto.createHash("md5 | sha1 | sha256 | sha512 | ripemd160")
	// hash.update(text)// 传输待加密文本text
	//   .digest("hex | base64")// 以16进制或"base64"方式输出，digest方法一旦调用，hash对象即被清空
	
	// 以密钥key和算法"md5"、"sha1"、"sha256"、"sha512"、"ripemd160"等生成hamc对象，密钥相关的哈希运算消息认证码
	//    加密后不可逆
	// 对称加密，服务器和客户端通过同样的key值进行加密解密
	// 不对称加密RSA、DSA、DH等，服务器和客户端各自拥有两个key，即公有密钥和私有密钥
	//    公有密钥加密过的数据，须私有密钥才能解开，反之亦然；私有密钥留为己用，公有密钥传输给另一方
	// SSL协议，位于应用层和TCP/IP之间；加密后从应用层流入TCP/IP，解密后从TCP/IP流入应用层；包含包协议和握手协议
	//    SSL握手过程即，通信双方通过不对称加密算法来协商好一个对称加密算法以及使用的key，该算法用于加密传输的数据
	//    参看https://cnodejs.org/topic/504061d7fef591855112bab5
	// const hamc = crypto.createHmac("md5 | sha1 | sha256 | sha512 | ripemd160",key)
	// hamc.update(text)// 传输待加密文本text
	//   .digest("hex | base64")// 以16进制或"base64"方式输出，digest方法一旦调用，hash对象即被清空
	var crypto = require("crypto")
	  
	// 参数keys数组形式设定加密密钥，取keys[0]；参数algorithm加密算法；参数encoding加密后文本输出密码
	function Keygrip(keys, algorithm, encoding) {
	  if (!algorithm) algorithm = "sha1";
	  if (!encoding) encoding = "base64";
	
	  // Keygrip函数作为构造函数调用
	  if (!(this instanceof Keygrip)) return new Keygrip(keys, algorithm, encoding)
	
	  if (!keys || !(0 in keys)) {
	    throw new Error("Keys must be provided.")
	  }
	
	  // 基于密钥key加密data并替换"/"、"+"、"="后输出
	  function sign(data, key) {
	    return crypto
	      .createHmac(algorithm, key)
	      .update(data).digest(encoding)
	      .replace(/\/|\+|=/g, function(x) {
	        return ({ "/": "_", "+": "-", "=": "" })[x]
	      })
	  }
	
	  // 即加密数据，基于密钥keys[0]加密data并替换"/"、"+"、"="后输出
	  this.sign = function(data){ return sign(data, keys[0]) }
	
	  // 即密文比较，判断data能否经keys密钥数组中的某个密钥加密成digest
	  this.verify = function(data, digest) {
	    return this.index(data, digest) > -1
	  }
	
	  // 即密钥比较，由keys密钥数组依次加密data，获取加密后文本与digest匹配的key值序号
	  this.index = function(data, digest) {
	    for (var i = 0, l = keys.length; i < l; i++) {
	      if (constantTimeCompare(digest, sign(data, keys[i]))) return i
	    }
	
	    return -1
	  }
	}
	
	Keygrip.sign = Keygrip.verify = Keygrip.index = function() {
	  throw new Error("Usage: require('keygrip')(<array-of-keys>)")
	}
	
	// 判断两个加密后的文案是否等值
	var constantTimeCompare = function(val1, val2){
	    if(val1 == null && val2 != null){
	        return false;
	    } else if(val2 == null && val1 != null){
	        return false;
	    } else if(val1 == null && val2 == null){
	        return true;
	    }
	
	    if(val1.length !== val2.length){
	        return false;
	    }
	
	    var matches = 1;
	
	    for(var i = 0; i < val1.length; i++){
	        matches &= (val1.charAt(i) === val2.charAt(i) ? 1 : 0); //Don't short circuit
	    }
	
	    return matches === 1;
	};
	
	module.exports = Keygrip
	
## 参考

[浅谈nodejs中的Crypto模块](https://cnodejs.org/topic/504061d7fef591855112bab5)

[详解Node.js API系列 Crypto加密模块](http://blog.csdn.net/youyudehexie/article/details/12040797)
