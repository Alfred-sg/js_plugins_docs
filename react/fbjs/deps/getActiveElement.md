# getActiveElement

getActiveElement()，输出页面中获得焦点的元素，或者document.body，或者null。

    'use strict';

    function getActiveElement(){
      if (typeof document === 'undefined') {
        return null;
      }
      try {
        return document.activeElement || document.body;
      } catch (e) {
        return document.body;
      }
    }

    module.exports = getActiveElement;