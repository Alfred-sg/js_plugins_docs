# xhrSimpleDataSerializer

xhrSimpleDataSerializer(data)，将数组或对象data转换为查询字符串输出，如key=value；实际查询字符串需要data[key]=value。

    'use strict';

    // 将数组或对象data转换为查询字符串输出，如key=value；实际查询字符串需要data[key]=value
    function xhrSimpleDataSerializer(data) {
      var uri = [];
      var key;
      for (key in data) {
        uri.push(encodeURIComponent(key) + '=' + encodeURIComponent(data[key]));
      }
      return uri.join('&');
    }

    module.exports = xhrSimpleDataSerializer;