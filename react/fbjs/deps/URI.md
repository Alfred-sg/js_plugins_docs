# URI

new URI(uri)，创建URI实例，toString方法获取uri。

    'use strict';

    function _classCallCheck(instance, Constructor) { if (!(instance instanceof Constructor)) { throw new TypeError("Cannot call a class as a function"); } }

    var URI = function () {
      function URI(uri) {
        _classCallCheck(this, URI);

        this._uri = uri;
      }

      URI.prototype.toString = function toString() {
        return this._uri;
      };

      return URI;
    }();

    module.exports = URI;