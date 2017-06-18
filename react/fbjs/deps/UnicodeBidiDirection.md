# UnicodeBidiDirection

## 使用

* NEUTRAL、LTR、RTL，获取"NEUTRAL"、"LTR"、"RTL"文本取向字符串。
* isStrong(dir)，判断dir是否"LTR"、"RTL"中的一个。
* getHTMLDir(dir)，获取小写形式的"ltr"或"rtl"。
* getHTMLDirIfDifferent(dir,otherDir)，当dir不等于otherDir时，以dir转化获取"ltr"或"rtl"。
* setGlobalDir、getGlobalDir、initGlobalDir设置、获取或初始化设置缓存变量globalDir，初始化默认值为"ltr"。

## 源码

    'use strict';
    
    var invariant = require('./invariant');
    
    var NEUTRAL = 'NEUTRAL'; // No strong direction
    var LTR = 'LTR'; // Left-to-Right direction
    var RTL = 'RTL'; // Right-to-Left direction
    
    var globalDir = null;
    
    function isStrong(dir) {
      return dir === LTR || dir === RTL;
    }
    
    function getHTMLDir(dir) {
      !isStrong(dir) ? process.env.NODE_ENV !== 'production' ? 
        invariant(false, '`dir` must be a strong direction to be converted to HTML Direction') 
        : invariant(false) : void 0;
      return dir === LTR ? 'ltr' : 'rtl';
    }
    
    function getHTMLDirIfDifferent(dir, otherDir) {
      !isStrong(dir) ? process.env.NODE_ENV !== 'production' ? 
        invariant(false, '`dir` must be a strong direction to be converted to HTML Direction') 
        : invariant(false) : void 0;
      !isStrong(otherDir) ? process.env.NODE_ENV !== 'production' ? 
        invariant(false, '`otherDir` must be a strong direction to be converted to HTML Direction') 
        : invariant(false) : void 0;
      return dir === otherDir ? null : getHTMLDir(dir);
    }
    
    function setGlobalDir(dir) {
      globalDir = dir;
    }
    
    function initGlobalDir() {
      setGlobalDir(LTR);
    }
    
    function getGlobalDir() {
      if (!globalDir) {
        this.initGlobalDir();
      }
      !globalDir ? process.env.NODE_ENV !== 'production' ? 
        invariant(false, 'Global direction not set.') : invariant(false) : void 0;
      return globalDir;
    }
    
    var UnicodeBidiDirection = {
      // Values
      NEUTRAL: NEUTRAL,
      LTR: LTR,
      RTL: RTL,
      // Helpers
      isStrong: isStrong,
      getHTMLDir: getHTMLDir,
      getHTMLDirIfDifferent: getHTMLDirIfDifferent,
      // Global Direction
      setGlobalDir: setGlobalDir,
      initGlobalDir: initGlobalDir,
      getGlobalDir: getGlobalDir
    };
    
    module.exports = UnicodeBidiDirection;
