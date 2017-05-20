# enumerate

## 接口

* enumerate["KIND_KEYS"|"KIND_VALUES"|"KIND_ENTRIES"]，获取迭代类型常数"keys"|"values"|"entries"。
* enumerate.keys(data)，获取键迭代器；字符串无，数组序号，对象键，或默认迭代器。
* enumerate.values(data)，获取值迭代器；字符串字符，数组项，对象值，或默认迭代器。
* enumerate.entries(data)，获取键、值迭代器；字符串无，数组序号加元素，对象键加值，或默认迭代器。
* enumerate.generic(object)，获取对象的属性、值迭代器。

## 源码

    'use strict';

    var _assign = require('object-assign');

    function _classCallCheck(instance, Constructor) { if (!(instance instanceof Constructor)) { throw new TypeError("Cannot call a class as a function"); } }

    var KIND_KEYS = 'keys';
    var KIND_VALUES = 'values';
    var KIND_ENTRIES = 'entries';

    // ArrayIterators.keys(arr)数组序号迭代器
    // ArrayIterators.values(arr)数组项迭代器
    // ArrayIterators.entries(arr)数组序号、数组项迭代器
    var ArrayIterators = function () {

      var hasNative = hasNativeIterator(Array);
      var ArrayIterator = void 0;

      if (!hasNative) {
        ArrayIterator = function () {
          function ArrayIterator(array, kind) {
            _classCallCheck(this, ArrayIterator);

            this._iteratedObject = array;
            this._kind = kind;
            this._nextIndex = 0;
          }

          ArrayIterator.prototype.next = function next() {
            if (this._iteratedObject == null) {
              return { value: undefined, done: true };
            }

            var array = this._iteratedObject;
            var len = this._iteratedObject.length;
            var index = this._nextIndex;
            var kind = this._kind;

            if (index >= len) {
              this._iteratedObject = undefined;
              return { value: undefined, done: true };
            }

            this._nextIndex = index + 1;

            if (kind === KIND_KEYS) {
              return { value: index, done: false };
            } else if (kind === KIND_VALUES) {
              return { value: array[index], done: false };
            } else if (kind === KIND_ENTRIES) {
              return { value: [index, array[index]], done: false };
            }
          };

          ArrayIterator.prototype[Symbol.iterator] = function () {
            return this;
          };

          return ArrayIterator;
        }();
      }

      return {
        keys: hasNative ? function (array) {
          return array.keys();
        } : function (array) {
          return new ArrayIterator(array, KIND_KEYS);
        },

        values: hasNative ? function (array) {
          return array.values();
        } : function (array) {
          return new ArrayIterator(array, KIND_VALUES);
        },

        entries: hasNative ? function (array) {
          return array.entries();
        } : function (array) {
          return new ArrayIterator(array, KIND_ENTRIES);
        }
      };
    }();

    // StringIterators.values(str)返回字符串迭代器
    var StringIterators = function () {

      var hasNative = hasNativeIterator(String);
      var StringIterator = void 0;

      if (!hasNative) {
        StringIterator = function () {
          function StringIterator(string) {
            _classCallCheck(this, StringIterator);

            this._iteratedString = string;// 待迭代的字符串
            this._nextIndex = 0;// next方法迭代位置
          }

          StringIterator.prototype.next = function next() {
            if (this._iteratedString == null) {
              return { value: undefined, done: true };
            }

            var index = this._nextIndex;
            var s = this._iteratedString;
            var len = s.length;

            if (index >= len) {
              this._iteratedString = undefined;
              return { value: undefined, done: true };
            }

            var ret = void 0;
            var first = s.charCodeAt(index);

            if (first < 0xD800 || first > 0xDBFF || index + 1 === len) {
              ret = s[index];
            } else {
              var second = s.charCodeAt(index + 1);
              // 单字节字符串
              if (second < 0xDC00 || second > 0xDFFF) {
                ret = s[index];
              // 双字节字符串
              } else {
                ret = s[index] + s[index + 1];
              }
            }

            this._nextIndex = index + ret.length;

            return { value: ret, done: false };
          };

          StringIterator.prototype[Symbol.iterator] = function () {
            return this;
          };

          return StringIterator;
        }();
      }

      return {
        keys: function keys() {
          throw TypeError('Strings default iterator doesn\'t implement keys.');
        },


        values: hasNative ? function (string) {
          return string[Symbol.iterator]();
        } : function (string) {
          return new StringIterator(string);
        },

        entries: function entries() {
          throw TypeError('Strings default iterator doesn\'t implement entries.');
        }
      };
    }();

    // 判断浏览器环境是否支持迭代器
    function hasNativeIterator(classObject) {
      return typeof classObject.prototype[Symbol.iterator] === 'function' && 
        typeof classObject.prototype.values === 'function' && 
        typeof classObject.prototype.keys === 'function' && 
        typeof classObject.prototype.entries === 'function';
    }

    // ObjectIterator(object,["keys"|"values"|"entries"])对象属性、值、属性及值迭代器
    var ObjectIterator = function () {
      function ObjectIterator(object, kind) {
        _classCallCheck(this, ObjectIterator);

        this._iteratedObject = object;
        this._kind = kind;
        this._keys = Object.keys(object);
        this._nextIndex = 0;
      }

      ObjectIterator.prototype.next = function next() {
        var len = this._keys.length;
        var index = this._nextIndex;
        var kind = this._kind;
        var key = this._keys[index];

        if (index >= len) {
          this._iteratedObject = undefined;
          return { value: undefined, done: true };
        }

        this._nextIndex = index + 1;

        if (kind === KIND_KEYS) {
          return { value: key, done: false };
        } else if (kind === KIND_VALUES) {
          return { value: this._iteratedObject[key], done: false };
        } else if (kind === KIND_ENTRIES) {
          return { value: [key, this._iteratedObject[key]], done: false };
        }
      };

      ObjectIterator.prototype[Symbol.iterator] = function () {
        return this;
      };

      return ObjectIterator;
    }();

    // GenericIterators.keys(object)属性迭代器
    // GenericIterators.values(object)值迭代器
    // GenericIterators.entries(object)属性、值迭代器
    var GenericIterators = {
      keys: function keys(object) {
        return new ObjectIterator(object, KIND_KEYS);
      },
      values: function values(object) {
        return new ObjectIterator(object, KIND_VALUES);
      },
      entries: function entries(object) {
        return new ObjectIterator(object, KIND_ENTRIES);
      }
    };

    // 根据kind类型["keys"|"values"|"entries"]返回数据object字符串、数组、对象的迭代器或其默认迭代器
    function enumerate(object, kind) {
      if (typeof object === 'string') {
        return StringIterators[kind || KIND_VALUES](object);
      } else if (Array.isArray(object)) {
        return ArrayIterators[kind || KIND_VALUES](object);

      // 默认迭代器
      } else if (object[Symbol.iterator]) {
        return object[Symbol.iterator]();

      } else {
        return GenericIterators[kind || KIND_ENTRIES](object);
      }
    }

    // 构建字符串、数组、对象的迭代器并返回；或获取默认的迭代器
    _assign(enumerate, {
      // 迭代类型["keys"|"values"|"entries"]
      KIND_KEYS: KIND_KEYS,
      KIND_VALUES: KIND_VALUES,
      KIND_ENTRIES: KIND_ENTRIES,

      // 键迭代器；字符串无，数组序号，对象键，或默认迭代器
      keys: function keys(object) {
        return enumerate(object, KIND_KEYS);
      },
      // 值迭代器；字符串字符，数组项，对象值，或默认迭代器
      values: function values(object) {
        return enumerate(object, KIND_VALUES);
      },
      // 键、值迭代器；字符串无，数组序号加元素，对象键加值，或默认迭代器
      entries: function entries(object) {
        return enumerate(object, KIND_ENTRIES);
      },

      // 对象的属性、值迭代器
      generic: GenericIterators.entries

    });

    module.exports = enumerate;
