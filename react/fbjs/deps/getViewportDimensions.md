# getViewportDimensions

getViewportDimensions()，获取页面的宽高，包含滚动条。
getViewportDimensions.withoutScrollbars()，获取页面的宽高，不包含滚动条。

    "use strict";

    function getViewportWidth() {
      var width = void 0;
      if (document.documentElement) {
        width = document.documentElement.clientWidth;
      }

      if (!width && document.body) {
        width = document.body.clientWidth;
      }

      return width || 0;
    } 

    function getViewportHeight() {
      var height = void 0;
      if (document.documentElement) {
        height = document.documentElement.clientHeight;
      }

      if (!height && document.body) {
        height = document.body.clientHeight;
      }

      return height || 0;
    }

    // 获取页面的宽高，window.innerWidth、window.innerHeight获取时包含滚动条
    function getViewportDimensions() {
      return {
        width: window.innerWidth || getViewportWidth(),
        height: window.innerHeight || getViewportHeight()
      };
    }

    // 获取页面的宽高，不包含滚动条
    getViewportDimensions.withoutScrollbars = function () {
      return {
        width: getViewportWidth(),
        height: getViewportHeight()
      };
    };

    module.exports = getViewportDimensions;