# CalendarPart

## 概述

绘制日期范围组件中，浮层内容节点。

## props属性

* direction，正向或反向显示前后日期，"left"日期范围选择器左面板显示前一个日期、右面板显示后一个日期

* 影响DateInput输入框的props属性：
* format，浮层内输入框格式化时间对象的形式
* locale，语言包，为浮层内日期选择组件中清除按钮设置文案、日期选择组件年日选择头
* prefixCls，样式类前缀
* disabledDate=(moment,props.value)=>{}，判断moment对象是否可用
* placeholder，浮层内输入框提示文案
* selectedValue，输入框的初始值
* onInputSelect(moment|null)，浮层内输入框值改变时执行函数
 
* 影响CalendarHeader年月点选组件的props属性：
* locale，语言包，为浮层内日期选择组件中清除按钮设置文案、日期选择组件年日选择头
* value，选中的日期、以及获取moment对象的语言包、以显示星期名
* prefixCls，样式类前缀
* showTimePicker，是否显示时分秒选择组件
* onValueChange(moment)，浮层内头部年、月点选节点改变值时执行函数，年份、月份选择面板点选时执行
 
* 影响timePicker时分秒组件的props属性：
* timePicker，以节点形式作为时分秒组件渲染，同时其props属性设置时分选择组件的props
* disabledTime=(value)=>{}，返回值作为props.defaultDisabledTime属性设置时分选择组件的props
* timePickerDisabledTime，设置时分选择组件的props
* onInputSelect(moment|null)，浮层内输入框值改变时执行函数
* value，选中的日期、以及获取moment对象的语言包、以显示星期名
 
* 影响DateTable日期选择面板的props属性：
* locale，语言包，为浮层内日期选择组件中清除按钮设置文案、日期选择组件年日选择头
* value，选中的日期、以及获取moment对象的语言包、以显示星期名
* prefixCls，样式类前缀
* showTimePicker，是否显示时分秒选择组件
* hoverValue，浮层内日期组件点选后的日期
* selectedValue，初始化选中的日期
* dateRender=(moment,props.value)，用于绘制浮层内日期渲染组件内容
* onSelect(moment)，浮层内日期选择组件点选时执行函数
* onDayHover(moment)，浮层内日期选择组件鼠标移入时执行函数
* disabledDate=(moment,props.value)=>{}，判断moment对象是否可用
* showWeekNumber，是否显示日期组件中的日期数

## 源码

    'use strict';
    
    Object.defineProperty(exports, "__esModule", {
      value: true
    });
    
    var _extends2 = require('babel-runtime/helpers/extends');
    var _extends3 = _interopRequireDefault(_extends2);
    
    var _react = require('react');
    var _react2 = _interopRequireDefault(_react);
    
    var _CalendarHeader = require('../calendar/CalendarHeader');
    var _CalendarHeader2 = _interopRequireDefault(_CalendarHeader);
    
    var _DateTable = require('../date/DateTable');
    var _DateTable2 = _interopRequireDefault(_DateTable);
    
    var _DateInput = require('../date/DateInput');
    var _DateInput2 = _interopRequireDefault(_DateInput);
    
    var _index = require('../util/index');
    
    function _interopRequireDefault(obj) { return obj && obj.__esModule ? obj : { "default": obj }; }
    
    var CalendarPart = _react2["default"].createClass({
      displayName: 'CalendarPart',
    
      propTypes: {
        value: _react.PropTypes.any,
        direction: _react.PropTypes.any,
        prefixCls: _react.PropTypes.any,
        locale: _react.PropTypes.any,
        selectedValue: _react.PropTypes.any,
        hoverValue: _react.PropTypes.any,
        showTimePicker: _react.PropTypes.bool,
        format: _react.PropTypes.any,
        placeholder: _react.PropTypes.any,
        disabledDate: _react.PropTypes.any,
        timePicker: _react.PropTypes.any,
        disabledTime: _react.PropTypes.any,
        onInputSelect: _react.PropTypes.func,
        timePickerDisabledTime: _react.PropTypes.object
      },
      render: function render() {
        var props = this.props;
        var value = props.value,
            direction = props.direction,
            prefixCls = props.prefixCls,
            locale = props.locale,
            selectedValue = props.selectedValue,
            format = props.format,
            placeholder = props.placeholder,
            disabledDate = props.disabledDate,
            timePicker = props.timePicker,
            disabledTime = props.disabledTime,
            timePickerDisabledTime = props.timePickerDisabledTime,
            showTimePicker = props.showTimePicker,
            hoverValue = props.hoverValue,
            onInputSelect = props.onInputSelect;
    
        var disabledTimeConfig = showTimePicker && disabledTime && timePicker ? 
          (0, _index.getTimeConfig)(selectedValue, disabledTime) : null;
        var rangeClassName = prefixCls + '-range';
        var newProps = {
          locale: locale,
          value: value,
          prefixCls: prefixCls,
          showTimePicker: showTimePicker
        };
        var index = direction === 'left' ? 0 : 1;
    
        // 时分选择组件
        var timePickerEle = showTimePicker && timePicker && _react2["default"].cloneElement(timePicker, (0, _extends3["default"])({
          showHour: true,
          showMinute: true,
          showSecond: true
        }, timePicker.props, disabledTimeConfig, timePickerDisabledTime, {
          onChange: onInputSelect,
          defaultOpenValue: value,
          value: selectedValue[index]
        }));
    
        return _react2["default"].createElement(
          'div',
          { className: rangeClassName + '-part ' + rangeClassName + '-' + direction },
    
          // 浮层内日期选择组件中的输入框
          _react2["default"].createElement(_DateInput2["default"], {
            format: format,// 将moment对象以format格式转化为字符串
            locale: locale,// 语言包，设置清除按钮的文案
            prefixCls: prefixCls,// 样式类前缀
            timePicker: timePicker,// 无用
            disabledDate: disabledDate,// (moment,props.value)=>{}判断时间对象是否有效
            placeholder: placeholder,// 提示文案
            disabledTime: disabledTime,// 无用
            value: value,// 无用
            showClear: false,// 是否显示清除按钮
            selectedValue: selectedValue[index],// 初始值
            onChange: onInputSelect// 输入框值改变时执行函数
          }),
    
          _react2["default"].createElement(
            'div',
            { style: { outline: 'none' } },
    
            // 浮层内日期选择组件头部年、月点选节点
            _react2["default"].createElement(_CalendarHeader2["default"], (0, _extends3["default"])({}, newProps, {
              enableNext: direction === 'right',// 下一年、下一月点选按钮是否显示
              enablePrev: direction === 'left',// 上一年、上一月点选按钮是否显示
              onValueChange: props.onValueChange// 改变选中的日期对象，年份、月份选择面板点选时执行
            })),
    
            // 显示时分秒选择组件
            showTimePicker ? _react2["default"].createElement(
              'div',
              { className: prefixCls + '-time-picker' },
              _react2["default"].createElement(
                'div',
                { className: prefixCls + '-time-picker-panel' },
                timePickerEle
              )
            ) : null,
    
            // 显示日期选择组件，包含星期显示组件、日期渲染组件
            _react2["default"].createElement(
              'div',
              { className: prefixCls + '-body' },
              _react2["default"].createElement(_DateTable2["default"], (0, _extends3["default"])({}, newProps, {
                hoverValue: hoverValue,// 浮层内日期组件点选后的日期
                selectedValue: selectedValue,// 初始化选中的日期
                dateRender: props.dateRender,// (moment,props.value)，用于绘制浮层内日期渲染组件内容
                onSelect: props.onSelect,// 日期选择组件点选时执行函数
                onDayHover: props.onDayHover,// 日期选择组件鼠标移入时执行函数
                disabledDate: disabledDate,// (moment,props.value)=>{}判断时间对象是否有效
                showWeekNumber: props.showWeekNumber// 是否显示日期组件中的日期数
              }))
            )
          )
        );
      }
    });
    
    // direction，正向或反向显示前后日期，"left"日期范围选择器左面板显示前一个日期、右面板显示后一个日期
    // 
    // 影响DateInput输入框的props属性：
    // format，浮层内输入框格式化时间对象的形式
    // locale，语言包，为浮层内日期选择组件中清除按钮设置文案、日期选择组件年日选择头
    // prefixCls，样式类前缀
    // disabledDate=(moment,props.value)=>{}，判断moment对象是否可用
    // placeholder，浮层内输入框提示文案
    // selectedValue，输入框的初始值
    // onInputSelect(moment|null)，浮层内输入框值改变时执行函数
    // 
    // 影响CalendarHeader年月点选组件的props属性：
    // locale，语言包，为浮层内日期选择组件中清除按钮设置文案、日期选择组件年日选择头
    // value，选中的日期、以及获取moment对象的语言包、以显示星期名
    // prefixCls，样式类前缀
    // showTimePicker，是否显示时分秒选择组件
    // onValueChange(moment)，浮层内头部年、月点选节点改变值时执行函数，年份、月份选择面板点选时执行
    // 
    // 影响timePicker时分秒组件的props属性：
    // timePicker，以节点形式作为时分秒组件渲染，同时其props属性设置时分选择组件的props
    // disabledTime=(value)=>{}，返回值作为props.defaultDisabledTime属性设置props
    // timePickerDisabledTime，设置时分选择组件的props
    // onInputSelect(moment|null)，浮层内输入框值改变时执行函数
    // value，选中的日期、以及获取moment对象的语言包、以显示星期名
    // 
    // 影响DateTable日期选择面板的props属性：
    // locale，语言包，为浮层内日期选择组件中清除按钮设置文案、日期选择组件年日选择头
    // value，选中的日期、以及获取moment对象的语言包、以显示星期名
    // prefixCls，样式类前缀
    // showTimePicker，是否显示时分秒选择组件
    // hoverValue，浮层内日期组件点选后的日期
    // selectedValue，初始化选中的日期
    // dateRender=(moment,props.value)，用于绘制浮层内日期渲染组件内容
    // onSelect(moment)，浮层内日期选择组件点选时执行函数
    // onDayHover(moment)，浮层内日期选择组件鼠标移入时执行函数
    // disabledDate=(moment,props.value)=>{}，判断moment对象是否可用
    // showWeekNumber，是否显示日期组件中的日期数
    exports["default"] = CalendarPart;
    module.exports = exports['default'];
