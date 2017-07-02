# getTestDocument::react-dom

重写document文档。

    'use strict';
    
    function getTestDocument(markup) {
      document.open();
      document.write(markup || '<!doctype html><html><meta charset=utf-8><title>test doc</title>');
      document.close();
      return document;
    }
    
    module.exports = getTestDocument;