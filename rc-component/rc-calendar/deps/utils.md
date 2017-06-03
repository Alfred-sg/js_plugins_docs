# utils

## 接口

* getTodayTime(moment)，以moment对象的locale()特定语言包获取今天。
* getTitleString(moment)，获取moment对象的时分秒。
* getTodayTimeStr(moment)，以moment对象的locale()特定语言包获取今天的时分秒。
* syncTime(moment1,moment2)，将moment实例moment1的时分秒属性拷贝给moment2。
* getTimeConfig(moment,(moment)=>{})，获取不可设置的时分秒数组{disabledHours:[],disabledMinutes:[],disabledSeconds:[]}，供时分秒选择面板判断用。
* isTimeValidByConfig(moment,{disabledHours,disabledMinutes,disabledSeconds})，以时分秒顺序判断moment对象是否可用。
* isTimeValid(moment,(moment)=>{})，通过次参函数判断moment对象是否可用。
* isAllowedDate(moment,(moment)=>{},(moment)=>{})，通过次参、三参函数判断moment对象是否可用。

## 源码

    'use strict';

    Object.defineProperty(exports, "__esModule", {
      value: true
    });

    var _extends2 = require('babel-runtime/helpers/extends');
    var _extends3 = _interopRequireDefault(_extends2);

    exports.getTodayTime = getTodayTime;
    exports.getTitleString = getTitleString;
    exports.getTodayTimeStr = getTodayTimeStr;
    exports.syncTime = syncTime;
    exports.getTimeConfig = getTimeConfig;
    exports.isTimeValidByConfig = isTimeValidByConfig;
    exports.isTimeValid = isTimeValid;
    exports.isAllowedDate = isAllowedDate;

    var _moment = require('moment');
    var _moment2 = _interopRequireDefault(_moment);

    function _interopRequireDefault(obj) { return obj && obj.__esModule ? obj : { "default": obj }; }

    var defaultDisabledTime = {
      disabledHours: function disabledHours() {
        return [];
      },
      disabledMinutes: function disabledMinutes() {
        return [];
      },
      disabledSeconds: function disabledSeconds() {
        return [];
      }
    };

    // 以特定语言获取今天
    function getTodayTime(value) {
      var today = (0, _moment2["default"])();
      today.locale(value.locale()).utcOffset(value.utcOffset());
      return today;
    }

    function getTitleString(value) {
      return value.year() + '-' + (value.month() + 1) + '-' + value.date();
    }

    function getTodayTimeStr(value) {
      var today = getTodayTime(value);
      return getTitleString(today);
    }

    // 将moment实例from的时分秒属性拷贝给to
    function syncTime(from, to) {
      to.hour(from.hour());
      to.minute(from.minute());
      to.second(from.second());
    }

    // 参数disabledTime以(moment)=>{}函数形式获取不可设置的时分秒数组，供时分秒选择面板判断用
    function getTimeConfig(value, disabledTime) {
      var disabledTimeConfig = disabledTime ? disabledTime(value) : {};
      disabledTimeConfig = (0, _extends3["default"])({}, defaultDisabledTime, disabledTimeConfig);
      return disabledTimeConfig;
    }

    // 以时分秒顺序判断moment对象value是否可用
    function isTimeValidByConfig(value, disabledTimeConfig) {
      var invalidTime = false;
      if (value) {
        var hour = value.hour();
        var minutes = value.minute();
        var seconds = value.second();
        var disabledHours = disabledTimeConfig.disabledHours();
        if (disabledHours.indexOf(hour) === -1) {
          var disabledMinutes = disabledTimeConfig.disabledMinutes(hour);
          if (disabledMinutes.indexOf(minutes) === -1) {
            var disabledSeconds = disabledTimeConfig.disabledSeconds(hour, minutes);
            invalidTime = disabledSeconds.indexOf(seconds) !== -1;
          } else {
            invalidTime = true;
          }
        } else {
          invalidTime = true;
        }
      }
      return !invalidTime;
    }

    // 通过参数disabledTime函数判断moment对象value是否可用
    function isTimeValid(value, disabledTime) {
      var disabledTimeConfig = getTimeConfig(value, disabledTime);
      return isTimeValidByConfig(value, disabledTimeConfig);
    }

    // 通过参数disabledDate函数、disabledTime函数判断moment对象value是否可用
    function isAllowedDate(value, disabledDate, disabledTime) {
      if (disabledDate) {
        if (disabledDate(value)) {
          return false;
        }
      }
      if (disabledTime) {
        if (!isTimeValid(value, disabledTime)) {
          return false;
        }
      }
      return true;
    }