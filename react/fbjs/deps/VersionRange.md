# VersionRange

VersionRange.contains(range,version)，判断实际版本号version是否在range范围中。

    'use strict';
    
    var invariant = require('./invariant');
    
    var componentRegex = /\./;
    var orRegex = /\|\|/;
    var rangeRegex = /\s+\-\s+/;
    var modifierRegex = /^(<=|<|=|>=|~>|~|>|)?\s*(.+)/;
    var numericRegex = /^(\d*)(.*)/;
    
    // 先比较range中含有"||"的版本号，再比较其他
    function checkOrExpression(range, version) {
      var expressions = range.split(orRegex);
    
      if (expressions.length > 1) {
        return expressions.some(function (range) {
          return VersionRange.contains(range, version);
        });
      } else {
        range = expressions[0].trim();
        return checkRangeExpression(range, version);
      }
    }
    
    // 独立比较range中含有"-"的版本号，再比较其他
    function checkRangeExpression(range, version) {
      var expressions = range.split(rangeRegex);
    
      !(expressions.length > 0 && expressions.length <= 2) ? 
        process.env.NODE_ENV !== 'production' ? invariant(false, 'the "-" operator expects exactly 2 operands') : invariant(false) : void 0;
    
      if (expressions.length === 1) {
        return checkSimpleExpression(expressions[0], version);
      } else {
        var startVersion = expressions[0],
            endVersion = expressions[1];
    
        !(isSimpleVersion(startVersion) && isSimpleVersion(endVersion)) ? process.env.NODE_ENV !== 'production' ? invariant(false, 'operands to the "-" operator must be simple (no modifiers)') : invariant(false) : void 0;
    
        return checkSimpleExpression('>=' + startVersion, version) && checkSimpleExpression('<=' + endVersion, version);
      }
    }
    
    // 比较range中不含"-"或"||"的版本号
    function checkSimpleExpression(range, version) {
      range = range.trim();
      if (range === '') {
        return true;
      }
    
      var versionComponents = version.split(componentRegex);
    
      var _getModifierAndCompon = getModifierAndComponents(range),
          modifier = _getModifierAndCompon.modifier,
          rangeComponents = _getModifierAndCompon.rangeComponents;
    
      switch (modifier) {
        case '<':
          return checkLessThan(versionComponents, rangeComponents);
        case '<=':
          return checkLessThanOrEqual(versionComponents, rangeComponents);
        case '>=':
          return checkGreaterThanOrEqual(versionComponents, rangeComponents);
        case '>':
          return checkGreaterThan(versionComponents, rangeComponents);
        case '~':
        case '~>':
          return checkApproximateVersion(versionComponents, rangeComponents);
        default:
          return checkEqual(versionComponents, rangeComponents);
      }
    }
    
    // 版本号比较
    function checkLessThan(a, b) {
      return compareComponents(a, b) === -1;
    }
    
    function checkLessThanOrEqual(a, b) {
      var result = compareComponents(a, b);
      return result === -1 || result === 0;
    }
    
    function checkEqual(a, b) {
      return compareComponents(a, b) === 0;
    }
    
    function checkGreaterThanOrEqual(a, b) {
      var result = compareComponents(a, b);
      return result === 1 || result === 0;
    }
    
    function checkGreaterThan(a, b) {
      return compareComponents(a, b) === 1;
    }
    
    // range中含有"-"版本号比较
    function checkApproximateVersion(a, b) {
      var lowerBound = b.slice();
      var upperBound = b.slice();
    
      if (upperBound.length > 1) {
        upperBound.pop();
      }
      var lastIndex = upperBound.length - 1;
      var numeric = parseInt(upperBound[lastIndex], 10);
      if (isNumber(numeric)) {
        upperBound[lastIndex] = numeric + 1 + '';
      }
    
      return checkGreaterThanOrEqual(a, lowerBound) && checkLessThan(a, upperBound);
    }
    
    // 将版本范围">= 1.2.3"转化成{modifier:">=",rangeComponents:[1, 2, 3]}输出
    function getModifierAndComponents(range) {
      var rangeComponents = range.split(componentRegex);
      var matches = rangeComponents[0].match(modifierRegex);
      !matches ? process.env.NODE_ENV !== 'production' ? 
        invariant(false, 'expected regex to match but it did not') : invariant(false) : void 0;
    
      return {
        modifier: matches[1],
        rangeComponents: [matches[2]].concat(rangeComponents.slice(1))
      };
    }
    
    function isNumber(number) {
      return !isNaN(number) && isFinite(number);
    }
    
    // 版本范围range是否精确匹配
    function isSimpleVersion(range) {
      return !getModifierAndComponents(range).modifier;
    }
    
    // 比较版本号，a、b为数组形式
    function compareComponents(a, b) {
      var _normalizeVersions = normalizeVersions(a, b),
          aNormalized = _normalizeVersions[0],
          bNormalized = _normalizeVersions[1];
    
      for (var i = 0; i < bNormalized.length; i++) {
        var result = compareNumeric(aNormalized[i], bNormalized[i]);
        if (result) {
          return result;
        }
      }
    
      return 0;
    }
    
    // 版本号拆解后进行比较
    function normalizeVersions(a, b) {
      a = a.slice();// 拷贝
      b = b.slice();
    
      zeroPad(a, b.length);
    
      // 匹配"x"或"*"位置双双赋予0，意味匹配
      for (var i = 0; i < b.length; i++) {
        var matches = b[i].match(/^[x*]$/i);
        if (matches) {
          b[i] = a[i] = '0';
    
          // "*"起始位后略过匹配
          if (matches[0] === '*' && i === b.length - 1) {
            for (var j = i; j < a.length; j++) {
              a[j] = '0';
            }
          }
        }
      }
    
      zeroPad(b, a.length);
    
      return [a, b];
    }
    
    // array中长度不到length部分补0
    function zeroPad(array, length) {
      for (var i = array.length; i < length; i++) {
        array[i] = '0';
      }
    }
    
    // 取a、b中的数值加以比较
    function compareNumeric(a, b) {
      var aPrefix = a.match(numericRegex)[1];
      var bPrefix = b.match(numericRegex)[1];
      var aNumeric = parseInt(aPrefix, 10);
      var bNumeric = parseInt(bPrefix, 10);
    
      if (isNumber(aNumeric) && isNumber(bNumeric) && aNumeric !== bNumeric) {
        return compare(aNumeric, bNumeric);
      } else {
        return compare(a, b);
      }
    }
    
    function compare(a, b) {
      !(typeof a === typeof b) ? process.env.NODE_ENV !== 'production' ? 
        invariant(false, '"a" and "b" must be of the same type') : invariant(false) : void 0;
    
      if (a > b) {
        return 1;
      } else if (a < b) {
        return -1;
      } else {
        return 0;
      }
    }
    
    var VersionRange = {
      /**
       * 版本号比较，range匹配条件，version实际版本号
       *
       *    version   Must match version exactly
       *    =version  Same as just version
       *    >version  Must be greater than version
       *    >=version Must be greater than or equal to version
       *    <version  Must be less than version
       *    <=version Must be less than or equal to version
       *    ~version  Must be at least version, but less than the next significant
       *              revision above version:
       *              "~1.2.3" is equivalent to ">= 1.2.3 and < 1.3"
       *    ~>version Equivalent to ~version
       *    1.2.x     Must match "1.2.x", where "x" is a wildcard that matches
       *              anything
       *    1.2.*     Similar to "1.2.x", but "*" in the trailing position is a
       *              "greedy" wildcard, so will match any number of additional
       *              components:
       *              "1.2.*" will match "1.2.1", "1.2.1.1", "1.2.1.1.1" etc
       *    *         Any version
       *    ""        (Empty string) Same as *
       *    v1 - v2   Equivalent to ">= v1 and <= v2"
       *    r1 || r2  Passes if either r1 or r2 are satisfied
       */
      contains: function contains(range, version) {
        return checkOrExpression(range.trim(), version.trim());
      }
    };
    
    module.exports = VersionRange;