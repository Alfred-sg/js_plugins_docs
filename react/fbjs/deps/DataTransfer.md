# DataTransfer

new DataTransfer(data)，资源类型判断或或获取资源。

    'use strict';
    
    function _classCallCheck(instance, Constructor) { 
      if (!(instance instanceof Constructor)) { 
        throw new TypeError("Cannot call a class as a function"); 
      } 
    }
    
    /**
     * Copyright (c) 2013-present, Facebook, Inc.
     * All rights reserved.
     *
     * This source code is licensed under the BSD-style license found in the
     * LICENSE file in the root directory of this source tree. An additional grant
     * of patent rights can be found in the PATENTS file in the same directory.
     *
     * @typechecks
     */
    
    var PhotosMimeType = require('./PhotosMimeType');
    
    // createArrayFromMixed(obj|arr|nodeList)，将单个对象、数组或伪数组转化成数组后输出。
    var createArrayFromMixed = require('./createArrayFromMixed');
    var emptyFunction = require('./emptyFunction');
    
    var CR_LF_REGEX = new RegExp('\r\n', 'g');
    var LF_ONLY = '\n';
    
    var RICH_TEXT_TYPES = {
      'text/rtf': 1,
      'text/html': 1
    };
    
    function getFileFromDataTransfer(item) {
      if (item.kind == 'file') {
        return item.getAsFile();
      }
    }
    
    var DataTransfer = function () {
      function DataTransfer(data) {
        _classCallCheck(this, DataTransfer);
    
        this.data = data;
    
        this.types = data.types ? createArrayFromMixed(data.types) : [];
      }
    
      // 判断是否富文本
      DataTransfer.prototype.isRichText = function isRichText() {
        if (this.getHTML() && this.getText()) {
          return true;
        }
    
        if (this.isImage()) {
          return false;
        }
    
        return this.types.some(function (type) {
          return RICH_TEXT_TYPES[type];
        });
      };
    
      // 获取文本数据
      DataTransfer.prototype.getText = function getText() {
        var text;
        if (this.data.getData) {
          if (!this.types.length) {
            text = this.data.getData('Text');
          } else if (this.types.indexOf('text/plain') != -1) {
            text = this.data.getData('text/plain');
          }
        }
        return text ? text.replace(CR_LF_REGEX, LF_ONLY) : null;
      };
    
      // 获取html数据
      DataTransfer.prototype.getHTML = function getHTML() {
        if (this.data.getData) {
          if (!this.types.length) {
            return this.data.getData('Text');
          } else if (this.types.indexOf('text/html') != -1) {
            return this.data.getData('text/html');
          }
        }
      };
    
      // 判断是否连接
      DataTransfer.prototype.isLink = function isLink() {
        return this.types.some(function (type) {
          return type.indexOf('Url') != -1 || type.indexOf('text/uri-list') != -1 || type.indexOf('text/x-moz-url');
        });
      };
    
      // 获取链接数据
      DataTransfer.prototype.getLink = function getLink() {
        if (this.data.getData) {
          if (this.types.indexOf('text/x-moz-url') != -1) {
            var url = this.data.getData('text/x-moz-url').split('\n');
            return url[0];
          }
          return this.types.indexOf('text/uri-list') != -1 ? this.data.getData('text/uri-list') : this.data.getData('url');
        }
    
        return null;
      };
    
      // 判断是否图片
      DataTransfer.prototype.isImage = function isImage() {
        var isImage = this.types.some(function (type) {
          return type.indexOf('application/x-moz-file') != -1;
        });
    
        if (isImage) {
          return true;
        }
    
        var items = this.getFiles();
        for (var i = 0; i < items.length; i++) {
          var type = items[i].type;
          if (!PhotosMimeType.isImage(type)) {
            return false;
          }
        }
    
        return true;
      };
    
      // 获取资源数
      DataTransfer.prototype.getCount = function getCount() {
        if (this.data.hasOwnProperty('items')) {
          return this.data.items.length;
        } else if (this.data.hasOwnProperty('mozItemCount')) {
          return this.data.mozItemCount;
        } else if (this.data.files) {
          return this.data.files.length;
        }
        return null;
      };
    
      // 获取文件
      DataTransfer.prototype.getFiles = function getFiles() {
        if (this.data.items) {
          return Array.prototype.slice.call(this.data.items).map(getFileFromDataTransfer).filter(emptyFunction.thatReturnsArgument);
        } else if (this.data.files) {
          return Array.prototype.slice.call(this.data.files);
        } else {
          return [];
        }
      };
    
      // 判断是否文件
      DataTransfer.prototype.hasFiles = function hasFiles() {
        return this.getFiles().length > 0;
      };
    
      return DataTransfer;
    }();
    
    module.exports = DataTransfer;