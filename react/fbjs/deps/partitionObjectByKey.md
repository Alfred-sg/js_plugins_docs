# partitionObjectByKey

partitionObjectByKey(source,whitelist)，借助partitionObject遍历source对象的属性，将包含在与不包含在白名单whitelist中的object属性分为两组，构成数组形式后输出。

    'use strict';

    var partitionObject = require('./partitionObject');

    // 以两元数组形式输出白名单中的属性、和不在白名单中的属性
    function partitionObjectByKey(source, whitelist) {
      return partitionObject(source, function (_, key) {
        return whitelist.has(key);
      });
    }

    module.exports = partitionObjectByKey;