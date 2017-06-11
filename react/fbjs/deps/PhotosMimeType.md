# PhotosMimeType

PhotosMimeType.isImage|isJpeg(mimetype)，用于判断媒体文件类型是否图片。

    'use strict';
    
    var PhotosMimeType = {
      isImage: function isImage(mimeString) {
        return getParts(mimeString)[0] === 'image';
      },
      isJpeg: function isJpeg(mimeString) {
        var parts = getParts(mimeString);
        return PhotosMimeType.isImage(mimeString) && (
        // see http://fburl.com/10972194
        parts[1] === 'jpeg' || parts[1] === 'pjpeg');
      }
    };
    
    function getParts(mimeString) {
      return mimeString.split('/');
    }
    
    module.exports = PhotosMimeType;