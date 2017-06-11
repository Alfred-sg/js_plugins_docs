# UserAgent

UserAgent.isBrowser | isBrowserArchitecture | isDevice | isEngine | isPlatform | isPlatformArchitecture，校验浏览器版本等信息是否匹配特定条件。

    'use strict';
    
    var UserAgentData = require('./UserAgentData');
    var VersionRange = require('./VersionRange');
    
    var mapObject = require('./mapObject');
    var memoizeStringOnly = require('./memoizeStringOnly');
    
    // 判断实际的name、version是否匹配query，normalizer用于转化query中版本号信息
    function compare(name, version, query, normalizer) {
      if (name === query) {
        return true;
      }
    
      if (!query.startsWith(name)) {
        return false;
      }
    
      var range = query.slice(name.length);
      if (version) {
        range = normalizer ? normalizer(range) : range;
        return VersionRange.contains(range, version);
      }
    
      return false;
    }
    
    function normalizePlatformVersion(version) {
      if (UserAgentData.platformName === 'Windows') {
        return version.replace(/^\s*NT/, '');
      }
    
      return version;
    }
    
    var UserAgent = {
      /**
       * 判断浏览器实际的版本号信息是否匹配query="Name [range expression]"
       *
       * - ACCESS NetFront
       * - AOL
       * - Amazon Silk
       * - Android
       * - BlackBerry
       * - BlackBerry PlayBook
       * - Chrome
       * - Chrome for iOS
       * - Chrome frame
       * - Facebook PHP SDK
       * - Facebook for iOS
       * - Firefox
       * - IE
       * - IE Mobile
       * - Mobile Safari
       * - Motorola Internet Browser
       * - Nokia
       * - Openwave Mobile Browser
       * - Opera
       * - Opera Mini
       * - Opera Mobile
       * - Safari
       * - UIWebView
       * - Unknown
       * - webOS
       * - etc...
       */
      isBrowser: function isBrowser(query) {
        return compare(UserAgentData.browserName, UserAgentData.browserFullVersion, query);
      },
    
      // 判断浏览器是32位还是64位，query="32" | "64"
      isBrowserArchitecture: function isBrowserArchitecture(query) {
        return compare(UserAgentData.browserArchitecture, null, query);
      },
    
      /**
       * 判断实际的设备信息是否匹配query="Name"
       *
       * - Kindle
       * - Kindle Fire
       * - Unknown
       * - iPad
       * - iPhone
       * - iPod
       * - etc...
       */
      isDevice: function isDevice(query) {
        return compare(UserAgentData.deviceName, null, query);
      },
    
      /**
       * 判断浏览器实际的引擎信息是否匹配query="Name [range expression]"
       *
       * - Gecko
       * - Presto
       * - Trident
       * - WebKit
       * - etc...
       */
      isEngine: function isEngine(query) {
        return compare(UserAgentData.engineName, UserAgentData.engineVersion, query);
      },
    
      /**
       * 
       * 判断实际的系统信息是否匹配query="Name [range expression]"
       *
       * - Android
       * - BlackBerry OS
       * - Java ME
       * - Linux
       * - Mac OS X
       * - Mac OS X Calendar
       * - Mac OS X Internet Account
       * - Symbian
       * - SymbianOS
       * - Windows
       * - Windows Mobile
       * - Windows Phone
       * - iOS
       * - iOS Facebook Integration Account
       * - iOS Facebook Social Sharing UI
       * - webOS
       * - Chrome OS
       * - etc...
       */
      isPlatform: function isPlatform(query) {
        return compare(UserAgentData.platformName, UserAgentData.platformFullVersion, query, normalizePlatformVersion);
      },
    
      // 判断操作系统是32位还是64位，query="32" | "64"
      isPlatformArchitecture: function isPlatformArchitecture(query) {
        return compare(UserAgentData.platformArchitecture, null, query);
      }
    };
    
    module.exports = mapObject(UserAgent, memoizeStringOnly);