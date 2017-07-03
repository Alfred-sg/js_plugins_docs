# escapeTextContentForBrowser::react-dom

escapeTextContentForBrowser(text)，布尔型和数值型转化为字符串后输出；字符串经html转码，逐个字符处理["'&<>]。

    'use strict';
    
    // 不含["'&<>]，跳过转码处理
    var matchHtmlRegExp = /["'&<>]/;
    
    // 逐个字符转码字符串
    function escapeHtml(string) {
      var str = '' + string;
      var match = matchHtmlRegExp.exec(str);
    
      if (!match) {
        return str;
      }
    
      var escape;
      var html = '';
      var index = 0;
      var lastIndex = 0;
    
      for (index = match.index; index < str.length; index++) {
        switch (str.charCodeAt(index)) {
          case 34:
            // "
            escape = '&quot;';
            break;
          case 38:
            // &
            escape = '&amp;';
            break;
          case 39:
            // '
            escape = '&#x27;'; // modified from escape-html; used to be '&#39'
            break;
          case 60:
            // <
            escape = '&lt;';
            break;
          case 62:
            // >
            escape = '&gt;';
            break;
          default:
            continue;
        }
    
        if (lastIndex !== index) {
          html += str.substring(lastIndex, index);
        }
    
        lastIndex = index + 1;
        html += escape;
      }
    
      return lastIndex !== index ? html + str.substring(lastIndex, index) : html;
    }
    
    // 布尔型和数值型转化为字符串后输出；字符串经html转码，逐个字符处理["'&<>]
    function escapeTextContentForBrowser(text) {
      if (typeof text === 'boolean' || typeof text === 'number') {
        return '' + text;
      }
      return escapeHtml(text);
    }
    
    module.exports = escapeTextContentForBrowser;
    