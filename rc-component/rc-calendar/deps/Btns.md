# Btns

## TodayButton

### 概述

绘制今天按钮。

### props属性

* prefixCls，样式类前缀
* locale，语言包，今天按钮文案
* text，自定义今天按钮文案
* timePicker，以节点形式作为时分秒组件渲染，同时其props属性设置时分选择组件的props
* value，选中的日期对象
* disabled，今天按钮是否可用
* disabledDate(moment,props.value)，判断moment日期节点是否不可点选
* onToday，今天按钮点击时执行函数

### 源码 

    'use strict';
    
    Object.defineProperty(exports, "__esModule", {
      value: true
    });
    exports["default"] = TodayButton;
    
    var _react = require('react');
    var _react2 = _interopRequireDefault(_react);
    
    var _util = require('../util/');
    
    function _interopRequireDefault(obj) { return obj && obj.__esModule ? obj : { "default": obj }; }
    
    function TodayButton(_ref) {
      var prefixCls = _ref.prefixCls,
          locale = _ref.locale,
          value = _ref.value,
          timePicker = _ref.timePicker,
          disabled = _ref.disabled,
          disabledDate = _ref.disabledDate,
          onToday = _ref.onToday,
          text = _ref.text;
    
      var localeNow = (!text && timePicker ? locale.now : text) || locale.today;
      var disabledToday = disabledDate && !(0, _util.isAllowedDate)((0, _util.getTodayTime)(value), disabledDate);
      var isDisabled = disabledToday || disabled;
      var disabledTodayClass = isDisabled ? prefixCls + '-today-btn-disabled' : '';
      return _react2["default"].createElement(
        'a',
        {
          className: prefixCls + '-today-btn ' + disabledTodayClass,
          role: 'button',
          onClick: isDisabled ? null : onToday,
          title: (0, _util.getTodayTimeStr)(value)
        },
        localeNow
      );
    }
    
    // prefixCls，样式类前缀
    // locale，语言包，今天按钮文案
    // text，自定义今天按钮文案
    // timePicker，以节点形式作为时分秒组件渲染，同时其props属性设置时分选择组件的props
    // value，选中的日期对象
    // disabled，今天按钮是否可用
    // disabledDate(moment,props.value)，判断moment日期节点是否不可点选
    // onToday，今天按钮点击时执行函数
    module.exports = exports['default'];



## TimePickerButton

### 概述

绘制日期、时间面板切换按钮。

### props属性

* prefixCls，样式类前缀
* locale，语言包，日期、时间面板切换按钮文案
* showTimePicker，时间面板显示中状态；若显示中，可切换为日期面板；若隐藏中，可切换为时间面板
* onOpenTimePicker，切换面板、显示时间面板时执行函数
* onCloseTimePicker，切换面板、隐藏时间面板时执行函数
* timePickerDisabled，日期、时间面板切换按钮是否可用

### 源码

    'use strict';
    
    Object.defineProperty(exports, "__esModule", {
      value: true
    });
    
    var _defineProperty2 = require('babel-runtime/helpers/defineProperty');
    var _defineProperty3 = _interopRequireDefault(_defineProperty2);
    
    exports["default"] = TimePickerButton;
    
    var _react = require('react');
    var _react2 = _interopRequireDefault(_react);
    
    var _classnames2 = require('classnames');
    var _classnames3 = _interopRequireDefault(_classnames2);
    
    function _interopRequireDefault(obj) { return obj && obj.__esModule ? obj : { "default": obj }; }
    
    function TimePickerButton(_ref) {
      var _classnames;
    
      var prefixCls = _ref.prefixCls,
          locale = _ref.locale,
          showTimePicker = _ref.showTimePicker,
          onOpenTimePicker = _ref.onOpenTimePicker,
          onCloseTimePicker = _ref.onCloseTimePicker,
          timePickerDisabled = _ref.timePickerDisabled;
    
      var className = (0, _classnames3["default"])((
        _classnames = {}, 
        (0, _defineProperty3["default"])(_classnames, prefixCls + '-time-picker-btn', true), 
        (0, _defineProperty3["default"])(_classnames, prefixCls + '-time-picker-btn-disabled', timePickerDisabled), 
        _classnames
      ));
      var onClick = null;
      if (!timePickerDisabled) {
        onClick = showTimePicker ? onCloseTimePicker : onOpenTimePicker;
      }
      return _react2["default"].createElement(
        'a',
        {
          className: className,
          role: 'button',
          onClick: onClick
        },
        showTimePicker ? locale.dateSelect : locale.timeSelect
      );
    }
    
    // prefixCls，样式类前缀
    // locale，语言包，日期、时间面板切换按钮文案
    // showTimePicker，时间面板显示中状态；若显示中，可切换为日期面板；若隐藏中，可切换为时间面板
    // onOpenTimePicker，切换面板、显示时间面板时执行函数
    // onCloseTimePicker，切换面板、隐藏时间面板时执行函数
    // timePickerDisabled，日期、时间面板切换按钮是否可用
    module.exports = exports['default'];



## OkButton

### 概述

绘制确认按钮。

### props属性

* prefixCls，样式类前缀
* locale，语言包，确认按钮文案
* okDisabled，确认按钮是否可用
* onOk，确认按钮点击时执行函数

### 源码

    "use strict";
    
    Object.defineProperty(exports, "__esModule", {
      value: true
    });
    exports["default"] = OkButton;
    
    var _react = require("react");
    var _react2 = _interopRequireDefault(_react);
    
    function _interopRequireDefault(obj) { return obj && obj.__esModule ? obj : { "default": obj }; }
    
    function OkButton(_ref) {
      var prefixCls = _ref.prefixCls,
          locale = _ref.locale,
          okDisabled = _ref.okDisabled,
          onOk = _ref.onOk;
    
      var className = prefixCls + "-ok-btn";
      if (okDisabled) {
        className += " " + prefixCls + "-ok-btn-disabled";
      }
      return _react2["default"].createElement(
        "a",
        {
          className: className,
          role: "button",
          onClick: okDisabled ? null : onOk
        },
        locale.ok
      );
    }
    
    // prefixCls，样式类前缀
    // locale，语言包，确认按钮文案
    // okDisabled，确认按钮是否可用
    // onOk，确认按钮点击时执行函数
    module.exports = exports['default'];
