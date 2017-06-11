# UnicodeUtils

## 使用

* UnicodeUtils.getCodePoints(str)，从字符串转化成unicode字符集数组。
* UnicodeUtils.getUTF16Length(str,pos)，BMP字符集[U+0000..U+D7FF]或[U+E000, U+FFFF]返回1，非BMP字符集[U+D800..U+DFFF]返回2。
* UnicodeUtils.hasSurrogateUnit(str)，判断是否含有非BMP字符集。
* UnicodeUtils.isCodeUnitInSurrogateRange(codeUnit)，判断字符codeUnit是否非BMP字符集[U+D800..U+DFFF]。
* UnicodeUtils.isSurrogatePair(str,index)，判断str[index]是否UTF16编码代理对，以SurrogatePair起始。
* UnicodeUtils.strlen(str)，BMP字符集直接返回长度，非BMP字符集两位起跳。
* UnicodeUtils.substring(str,start,end)，BMP字符集直接以参数截取字符，非BMP字符集两位起跳。
* UnicodeUtils.substr(str,start,length)， BMP字符集直接以参数截取字符，非BMP字符集两位起跳。

## 源码

    'use strict';
    
    var invariant = require('./invariant');
    
    var SURROGATE_HIGH_START = 0xD800;
    var SURROGATE_HIGH_END = 0xDBFF;
    var SURROGATE_LOW_START = 0xDC00;
    var SURROGATE_LOW_END = 0xDFFF;
    var SURROGATE_UNITS_REGEX = /[\uD800-\uDFFF]/;
    
    // 判断字符codeUnit是否非BMP字符集[U+D800..U+DFFF]
    function isCodeUnitInSurrogateRange(codeUnit) {
      return SURROGATE_HIGH_START <= codeUnit && codeUnit <= SURROGATE_LOW_END;
    }
    
    // 判断str[index]是否UTF16编码代理对，以SurrogatePair起始
    function isSurrogatePair(str, index) {
      !(0 <= index && index < str.length) ? process.env.NODE_ENV !== 'production' ? 
        invariant(false, 'isSurrogatePair: Invalid index %s for string length %s.', index, str.length) : 
        invariant(false) : void 0;
      if (index + 1 === str.length) {
        return false;
      }
      var first = str.charCodeAt(index);
      var second = str.charCodeAt(index + 1);
      return SURROGATE_HIGH_START <= first && first <= SURROGATE_HIGH_END && SURROGATE_LOW_START <= second && second <= SURROGATE_LOW_END;
    }
    
    // 判断是否含有非BMP字符集
    function hasSurrogateUnit(str) {
      return SURROGATE_UNITS_REGEX.test(str);
    }
    
    // BMP字符集[U+0000..U+D7FF]或[U+E000, U+FFFF]返回1，非BMP字符集[U+D800..U+DFFF]返回2
    function getUTF16Length(str, pos) {
      return 1 + isCodeUnitInSurrogateRange(str.charCodeAt(pos));
    }
    
    // BMP字符集直接返回长度，非BMP字符集两位起跳
    function strlen(str) {
      if (!hasSurrogateUnit(str)) {
        return str.length;
      }
    
      var len = 0;
      for (var pos = 0; pos < str.length; pos += getUTF16Length(str, pos)) {
        len++;
      }
      return len;
    }
    
    // BMP字符集直接以参数截取字符，非BMP字符集两位起跳
    function substr(str, start, length) {
      start = start || 0;
      length = length === undefined ? Infinity : length || 0;
    
      if (!hasSurrogateUnit(str)) {
        return str.substr(start, length);
      }
    
      var size = str.length;
      if (size <= 0 || start > size || length <= 0) {
        return '';
      }
    
      var posA = 0;
      if (start > 0) {
        for (; start > 0 && posA < size; start--) {
          posA += getUTF16Length(str, posA);
        }
        if (posA >= size) {
          return '';
        }
      } else if (start < 0) {
        for (posA = size; start < 0 && 0 < posA; start++) {
          posA -= getUTF16Length(str, posA - 1);
        }
        if (posA < 0) {
          posA = 0;
        }
      }
    
      var posB = size;
      if (length < size) {
        for (posB = posA; length > 0 && posB < size; length--) {
          posB += getUTF16Length(str, posB);
        }
      }
    
      return str.substring(posA, posB);
    }
    
    // BMP字符集直接以参数截取字符，非BMP字符集两位起跳
    function substring(str, start, end) {
      start = start || 0;
      end = end === undefined ? Infinity : end || 0;
    
      if (start < 0) {
        start = 0;
      }
      if (end < 0) {
        end = 0;
      }
    
      var length = Math.abs(end - start);
      start = start < end ? start : end;
      return substr(str, start, length);
    }
    
    // 从字符串转化成unicode字符集数组
    function getCodePoints(str) {
      var codePoints = [];
      for (var pos = 0; pos < str.length; pos += getUTF16Length(str, pos)) {
        codePoints.push(str.codePointAt(pos));
      }
      return codePoints;
    }
    
    var UnicodeUtils = {
      getCodePoints: getCodePoints,// 从字符串转化成unicode字符集数组
      getUTF16Length: getUTF16Length,// BMP字符集[U+0000..U+D7FF]或[U+E000, U+FFFF]返回1，非BMP字符集[U+D800..U+DFFF]返回2
      hasSurrogateUnit: hasSurrogateUnit,// 判断是否含有非BMP字符集
      isCodeUnitInSurrogateRange: isCodeUnitInSurrogateRange,// 判断字符codeUnit是否非BMP字符集[U+D800..U+DFFF]
      isSurrogatePair: isSurrogatePair,// 判断str[index]是否UTF16编码代理对，以SurrogatePair起始
      strlen: strlen,// BMP字符集直接返回长度，非BMP字符集两位起跳
      substring: substring,// BMP字符集直接以参数截取字符，非BMP字符集两位起跳
      substr: substr// BMP字符集直接以参数截取字符，非BMP字符集两位起跳
    };
    
    module.exports = UnicodeUtils;
