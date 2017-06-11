# UserAgentData

UserAgentData.browserArchitecture | browserFullVersion | browserMinorVersion | browserName | browserVersion | deviceName | engineName | engineVersion | platformArchitecture | platformName | platformVersion | platformFullVersion，通过ua-parser-js模块解析window.navigator.userAgent，获取系统、设备、浏览器、cpu、引擎等相关数据。

    'use strict';
    
    // new UAParser().getResult()，通过window.navigator.userAgent获取系统、设备、浏览器、cpu、引擎等相关数据
    // { 
    //   ua: "", 
    //   browser: { name: "", version: "" },
    //   engine: { name: "", version: "" },
    //   os: { name: "", version: "" },
    //   device: { model: "", type: "", vendor: "" },
    //   cpu: { architecture: "" }
    // }
    var UAParser = require('ua-parser-js');
    
    var UNKNOWN = 'Unknown';
    
    var PLATFORM_MAP = {
      'Mac OS': 'Mac OS X'
    };
    
    function convertPlatformName(name) {
      return PLATFORM_MAP[name] || name;
    }
    
    function getBrowserVersion(version) {
      if (!version) {
        return {
          major: '',
          minor: ''
        };
      }
      var parts = version.split('.');
      return {
        major: parts[0],
        minor: parts[1]
      };
    }
    
    var parser = new UAParser();
    var results = parser.getResult();
    
    // 通过window.navigator.userAgent获取系统、设备、浏览器、cpu、引擎等相关数据
    var browserVersionData = getBrowserVersion(results.browser.version);
    var uaData = {
      browserArchitecture: results.cpu.architecture || UNKNOWN,
      browserFullVersion: results.browser.version || UNKNOWN,
      browserMinorVersion: browserVersionData.minor || UNKNOWN,
      browserName: results.browser.name || UNKNOWN,
      browserVersion: results.browser.major || UNKNOWN,
      deviceName: results.device.model || UNKNOWN,
      engineName: results.engine.name || UNKNOWN,
      engineVersion: results.engine.version || UNKNOWN,
      platformArchitecture: results.cpu.architecture || UNKNOWN,
      platformName: convertPlatformName(results.os.name) || UNKNOWN,
      platformVersion: results.os.version || UNKNOWN,
      platformFullVersion: results.os.version || UNKNOWN
    };
    
    module.exports = uaData;