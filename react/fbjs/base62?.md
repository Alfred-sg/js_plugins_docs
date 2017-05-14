# base62

base62(num)，将参数num转化为base62字符串并返回。

    'use strict';

    var BASE62 = '0123456789abcdefghijklmnopqrstuvwxyzABCDEFGHIJKLMNOPQRSTUVWXYZ';

    /**
     * 将数值转化为62进制后，再转化成base62编码
     * 
     * @param  {number}
     * @return {[type]}
     */
    function base62(number) {
      if (!number) {
        return '0';
      }
      var string = '';
      while (number > 0) {
        string = BASE62[number % 62] + string;
        number = Math.floor(number / 62);
      }
      return string;
    }

    module.exports = base62;

    // 附注

    // 实际的base62编码，不支持中文

    let Base62 = 'vPh7zZwA2LyU4bGq5tcVfIMxJi6XaSoK9CNp0OWljYTHQ8REnmu31BrdgeDkFs';
      
    let log10 = (x) => Math.log(x)/Math.log(10);

    // 将数值编码成base62
    let encode = (str) => {
      let out = '';
      let i = Math.floor(log10(str) / log10(62));

      while( i > 0 ){
        let a = Math.floor(str / Math.pow(62, t));
        out += Base62[a];
        str -= (a * Math.pow(62, t));
        i--;
      };

      return out;
    };

    // 将base62字符串解码成数值
    let decode = (str) => {
      let out = 0;
      let len = str.length - 1;
      let i = 0;

      while( i<=len ){
        out += Base62.indexOf(str.substr(i,1)) * Math.pow(62, len - i); 
        i++;
      };

      return out;
    };

    export default {
      encode,
      decode
    }
     

    // base64编码，支持中文；欠缺领会？？？

    let Base64 = "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz0123456789+/",
      unicodeStr256 = '',// 由数值1-255转化成unicode字符串拼接而成
      base64IndexArr = [],// unicode字符串在Base64字符串中的序号；不匹配为-1
      arr256 = [];// 1-255数组

    for(let i = 0; i < 256; i++) {
      var unicodeStrFromNum = String.fromCharCode(i);
      unicodeStr256 += unicodeStrFromNum;
      arr256[i] = i;
      base64IndexArr[i] = Base64.indexOf(unicodeStrFromNum);
    }

    let UTF8 = {
      encode: (strUni) => {
        let strUtf = strUni.replace(/[\u0080-\u07ff]/g, (c) => {
          // 转化为2字节unicode编码字符串
          // str.charCodeAt(i) 返回指定字符位置的Unicode编码
          // 将unicode编码\u0080-\u07ff转化成数值128-2047，即0000000010000000-0000011111111111
          let cc = c.charCodeAt(0);
          // String.fromCharCode(n1, n2, ..., nX) 将unicodeb编码转化为字符串输出
          // 0x0080-0x07ff右移6位后，返回0x0002-0x001f；0x0002-0x001f同0xc0按位或，返回0x00c0-0x00df
          // 0x0080-0x07ff同0x3f安位与，返回0x0000-0x003f；0x0000-0x003f同0x80安位或，返回0x0080-0x00bf
          return String.fromCharCode(0xc0 | cc >> 6, 0x80 | cc & 0x3f);
        }).replace(/[\u0800-\uffff]/g, (c) => {
          // 转化为3字节unicode编码字符串
          // 将unicode编码\u0800--\uffff转化成数值2048-65535
          let cc = c.charCodeAt(0);
          // 0x0800-0xffff右移12位后，返回0x0000-0x000f；0x0000-0x000f同0xe0按位或，返回0x00e0-0x00ef
          // 0x0800-0xffff右移6位后，返回0x0010-0x003f；0x0010-0x003f同0x3F按位与，返回0x0010-0x003f
          //    0x0010-0x003f同0x80按位或，返回0x0080-0x00bf
          // 0x0800-0xffff同0x3f安位与，返回0x0000-0x003f；0x0000-0x003f同0x80按位或，返回0x0080-0x003f
          return String.fromCharCode(0xe0 | cc >> 12, 0x80 | cc >> 6 & 0x3F, 0x80 | cc & 0x3f);
        });

        return strUtf;
      },

      decode: function(strUtf) {
        var strUni = strUtf.replace(/[\u00e0-\u00ef][\u0080-\u00bf][\u0080-\u00bf]/g, (c) => { 
          // 3字节字符串解码；先于2字节字符串解码，因2字节字符串易被当做3字节字符串
          var cc = ((c.charCodeAt(0) & 0x0f) << 12) | ((c.charCodeAt(1) & 0x3f) << 6) | (c.charCodeAt(2) & 0x3f);
          return String.fromCharCode(cc);
        }).replace(/[\u00c0-\u00df][\u0080-\u00bf]/g, (c) => { 
          // 2字节字符串解码
          let cc = (c.charCodeAt(0) & 0x1f) << 6 | c.charCodeAt(1) & 0x3f;
          return String.fromCharCode(cc);
        });

        return strUni;
      }
    };

    function code(s, discard, alpha, beta, w1, w2) {
      s = String(s);

      let buffer = 0,
        len = s.length,
        result = '',
        bitsInBuffer = 0;

      for(let i = 0; i < len; i++) {
        let c = s.charCodeAt(i);
        c = c < 256 ? alpha[c] : -1;

        buffer = (buffer << w1) + c;
        bitsInBuffer += w1;

        while(bitsInBuffer >= w2) {
          bitsInBuffer -= w2;
          let tmp = buffer >> bitsInBuffer;
          result += beta.charAt(tmp);
          buffer ^= tmp << bitsInBuffer;
        }
      }

      if(!discard && bitsInBuffer > 0) result += beta.charAt(buffer << (w2 - bitsInBuffer));

      return result;
    }

    let encode = (plain, utf8encode) => {
      plain = utf8encode ? UTF8.encode(plain) : plain;
      plain = code(plain, false, arr256, Base64, 8, 6);
      return plain + '===='.slice((plain.length % 4) || 4);
    };

    let decode = (coded, utf8decode) => {
      coded = coded.replace(/[^A-Za-z0-9\+\/\=]/g, "");
      coded = String(coded).split('=');
      let i = coded.length;
      do {--i;
        coded[i] = code(coded[i], true, base64IndexArr, unicodeStr256, 6, 8);
      } while (i > 0);
      coded = coded.join('');
      return utf8decode ? UTF8.decode(coded) : coded;
    };

    export default {
      encode,
      decode
    }