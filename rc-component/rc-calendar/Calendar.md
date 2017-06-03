# Calendar

## 概述

绘制日期单选组件。

## props属性

* locale，语言包
* prefixCls，样式类前缀
* className，样式类
* style，样式
* visible，浮层日期选择面板是否显示
* value，用于构成选中日期的时分秒，优先级高于state.defaultValue
* defaultValue，用于构成选中日期的时分秒
* selectedValue，用于构成选中日期的年月日，优先级高于state.defaultSelectedValue
* defaultSelectedValue，用于构成选中日期的年月日
* format，输入框格式化日期对象，优先级高于locale.dateTimeFormat或locale.dateFormat
* renderSidebar，绘制自定义侧边栏
* showToday，显示今天按钮
* onClear，清空按钮点击时触发执行回调函数
* showOk，显示确认按钮
* onOk，确认按钮点击时触发执行回调函数
* onKeyDown，键盘按键触发执行回调函数
* onChange，时间选择面板、键盘按键时执行回调
* onSelect，浮层内日期选择面板点击、今天按钮点击、清空按钮点击、浮层内输入框值改变、键盘enter键点击时执行回调

* 影响timePicker时间选择面板的props属性：
* timePicker，浮层内时间选择面板组件

* 影响dateInput浮层内输入框的props属性：
* showDateInput，是否显示输入框
* dateInputPlaceholder，输入框的文案提示

* 影响CalendarFooter浮层内尾部的props属性：
* renderFooter，构成尾部除按钮外其他节点

## 源码

    'use strict';
    
    Object.defineProperty(exports, "__esModule", {
      value: true
    });
    
    var _extends2 = require('babel-runtime/helpers/extends');
    var _extends3 = _interopRequireDefault(_extends2);
    
    var _react = require('react');
    var _react2 = _interopRequireDefault(_react);
    
    var _reactDom = require('react-dom');
    var _reactDom2 = _interopRequireDefault(_reactDom);
    
    var _KeyCode = require('rc-util/lib/KeyCode');
    var _KeyCode2 = _interopRequireDefault(_KeyCode);
    
    var _DateTable = require('./date/DateTable');
    var _DateTable2 = _interopRequireDefault(_DateTable);
    
    var _CalendarHeader = require('./calendar/CalendarHeader');
    var _CalendarHeader2 = _interopRequireDefault(_CalendarHeader);
    
    var _CalendarFooter = require('./calendar/CalendarFooter');
    var _CalendarFooter2 = _interopRequireDefault(_CalendarFooter);
    
    var _CalendarMixin = require('./mixin/CalendarMixin');
    var _CalendarMixin2 = _interopRequireDefault(_CalendarMixin);
    
    var _CommonMixin = require('./mixin/CommonMixin');
    var _CommonMixin2 = _interopRequireDefault(_CommonMixin);
    
    var _DateInput = require('./date/DateInput');
    var _DateInput2 = _interopRequireDefault(_DateInput);
    
    var _index = require('./util/index');
    
    function _interopRequireDefault(obj) { return obj && obj.__esModule ? obj : { "default": obj }; }
    
    function noop() {}
    
    function goStartMonth() {
      var next = this.state.value.clone();
      next.startOf('month');
      this.setValue(next);
    }
    
    function goEndMonth() {
      var next = this.state.value.clone();
      next.endOf('month');
      this.setValue(next);
    }
    
    function goTime(direction, unit) {
      var next = this.state.value.clone();
      next.add(direction, unit);
      this.setValue(next);
    }
    
    function goMonth(direction) {
      return goTime.call(this, direction, 'months');
    }
    
    function goYear(direction) {
      return goTime.call(this, direction, 'years');
    }
    
    function goWeek(direction) {
      return goTime.call(this, direction, 'weeks');
    }
    
    function goDay(direction) {
      return goTime.call(this, direction, 'days');
    }
    
    var Calendar = _react2["default"].createClass({
      displayName: 'Calendar',
    
      propTypes: {
        disabledDate: _react.PropTypes.func,
        disabledTime: _react.PropTypes.any,
        value: _react.PropTypes.object,
        selectedValue: _react.PropTypes.object,
        defaultValue: _react.PropTypes.object,
        className: _react.PropTypes.string,
        locale: _react.PropTypes.object,
        showWeekNumber: _react.PropTypes.bool,
        style: _react.PropTypes.object,
        showToday: _react.PropTypes.bool,
        showDateInput: _react.PropTypes.bool,
        visible: _react.PropTypes.bool,
        onSelect: _react.PropTypes.func,
        onOk: _react.PropTypes.func,
        showOk: _react.PropTypes.bool,
        prefixCls: _react.PropTypes.string,
        onKeyDown: _react.PropTypes.func,
        timePicker: _react.PropTypes.element,
        dateInputPlaceholder: _react.PropTypes.any,
        onClear: _react.PropTypes.func,
        onChange: _react.PropTypes.func,
        renderFooter: _react.PropTypes.func,
        renderSidebar: _react.PropTypes.func
      },
    
      mixins: [_CommonMixin2["default"], _CalendarMixin2["default"]],
    
      getDefaultProps: function getDefaultProps() {
        return {
          showToday: true,
          showDateInput: true,
          timePicker: null,
          onOk: noop
        };
      },
      getInitialState: function getInitialState() {
        return {
          showTimePicker: false
        };
      },
    
      // 键盘按键执行函数
      onKeyDown: function onKeyDown(event) {
        if (event.target.nodeName.toLowerCase() === 'input') {
          return undefined;
        }
        var keyCode = event.keyCode;
        // mac
        var ctrlKey = event.ctrlKey || event.metaKey;
        switch (keyCode) {
          case _KeyCode2["default"].DOWN:
            goWeek.call(this, 1);
            event.preventDefault();
            return 1;
          case _KeyCode2["default"].UP:
            goWeek.call(this, -1);
            event.preventDefault();
            return 1;
          case _KeyCode2["default"].LEFT:
            if (ctrlKey) {
              goYear.call(this, -1);
            } else {
              goDay.call(this, -1);
            }
            event.preventDefault();
            return 1;
          case _KeyCode2["default"].RIGHT:
            if (ctrlKey) {
              goYear.call(this, 1);
            } else {
              goDay.call(this, 1);
            }
            event.preventDefault();
            return 1;
          case _KeyCode2["default"].HOME:
            goStartMonth.call(this);
            event.preventDefault();
            return 1;
          case _KeyCode2["default"].END:
            goEndMonth.call(this);
            event.preventDefault();
            return 1;
          case _KeyCode2["default"].PAGE_DOWN:
            goMonth.call(this, 1);
            event.preventDefault();
            return 1;
          case _KeyCode2["default"].PAGE_UP:
            goMonth.call(this, -1);
            event.preventDefault();
            return 1;
          case _KeyCode2["default"].ENTER:
            this.onSelect(this.state.value, {
              source: 'keyboard'
            });
            event.preventDefault();
            return 1;
          default:
            this.props.onKeyDown(event);
            return 1;
        }
      },
    
      // 清空按钮点击时执行函数
      onClear: function onClear() {
        this.onSelect(null);
        this.props.onClear();
      },
    
      // 确认按钮点击时执行函数
      onOk: function onOk() {
        var selectedValue = this.state.selectedValue;
    
        if (this.isAllowedDate(selectedValue)) {
          this.props.onOk(selectedValue);
        }
      },
    
      // 浮层内输入框值改变时触发执行
      onDateInputChange: function onDateInputChange(value) {
        this.onSelect(value, {
          source: 'dateInput'
        });
      },
    
      // 日期选择面板点击时执行函数
      onDateTableSelect: function onDateTableSelect(value) {
        this.onSelect(value);
      },
    
      // 今天按钮点击时触发执行
      onToday: function onToday() {
        var value = this.state.value;
    
        var now = (0, _index.getTodayTime)(value);
        this.onSelect(now, {
          source: 'todayButton'
        });
      },
    
      // 获取外层包裹元素节点
      getRootDOMNode: function getRootDOMNode() {
        return _reactDom2["default"].findDOMNode(this);
      },
    
      // 显示隐藏时间选择面板
      openTimePicker: function openTimePicker() {
        this.setState({
          showTimePicker: true
        });
      },
      closeTimePicker: function closeTimePicker() {
        this.setState({
          showTimePicker: false
        });
      },
    
      render: function render() {
        var props = this.props;
        var locale = props.locale,
            prefixCls = props.prefixCls,
            disabledDate = props.disabledDate,
            dateInputPlaceholder = props.dateInputPlaceholder,
            timePicker = props.timePicker,
            disabledTime = props.disabledTime;
    
        var state = this.state;
        var value = state.value,
            selectedValue = state.selectedValue,
            showTimePicker = state.showTimePicker;
    
        var disabledTimeConfig = showTimePicker && disabledTime && timePicker ? 
          (0, _index.getTimeConfig)(selectedValue, disabledTime) : null;
    
        var timePickerEle = timePicker && showTimePicker ? _react2["default"].cloneElement(timePicker, (0, _extends3["default"])({
          showHour: true,
          showSecond: true,
          showMinute: true
        }, timePicker.props, disabledTimeConfig, {
          onChange: this.onDateInputChange,
          defaultOpenValue: value,
          value: selectedValue,
          disabledTime: disabledTime
        })) : null;
    
        var dateInputElement = props.showDateInput ? _react2["default"].createElement(_DateInput2["default"], {
          ref: 'dateInput',
          format: this.getFormat(),// 将moment对象以format格式转化为字符串
          key: 'date-input',
          value: value,// 无用
          locale: locale,// 语言包，设置清除按钮的文案
          placeholder: dateInputPlaceholder,// 提示文案
          showClear: true,// 是否显示清除按钮
          disabledTime: disabledTime,// 无用
          disabledDate: disabledDate,// (moment,props.value)=>{}判断时间对象是否有效
          onClear: this.onClear,// 清除按钮点击时执行函数
          prefixCls: prefixCls,// 提示文案
          selectedValue: selectedValue,// 初始值
          onChange: this.onDateInputChange// 输入框值改变时执行函数
        }) : null;
    
        var children = [props.renderSidebar(), _react2["default"].createElement(
          'div',
          { className: prefixCls + '-panel', key: 'panel' },
          dateInputElement,
          _react2["default"].createElement(
            'div',
            { className: prefixCls + '-date-panel' },
    
            _react2["default"].createElement(_CalendarHeader2["default"], {
              locale: locale,// 语言包
              onValueChange: this.setValue,// 改变选中的日期对象，年份、月份选择面板点选时执行
              value: value,// 影响选中日期的时分秒
              showTimePicker: showTimePicker,// 是否显示事件选择面板
              prefixCls: prefixCls// 样式类前缀
            }),
    
            timePicker && showTimePicker ? _react2["default"].createElement(
              'div',
              { className: prefixCls + '-time-picker' },
              _react2["default"].createElement(
                'div',
                { className: prefixCls + '-time-picker-panel' },
                timePickerEle
              )
            ) : null,
    
            _react2["default"].createElement(
              'div',
              { className: prefixCls + '-body' },
              _react2["default"].createElement(_DateTable2["default"], {
                locale: locale,// 语言包
                value: value,// 影响选中日期的时分秒
                selectedValue: selectedValue,// 初始化选中的日期
                prefixCls: prefixCls,// 样式类前缀
                dateRender: props.dateRender,// (moment,props.value)，用于绘制浮层内日期渲染组件内容
                onSelect: this.onDateTableSelect,// 日期选择组件点选时执行函数
                disabledDate: disabledDate,// (moment,props.value)=>{}判断时间对象是否有效
                showWeekNumber: props.showWeekNumber// 是否显示日期组件中的日期数
              })
            ),
            _react2["default"].createElement(_CalendarFooter2["default"], {
              showOk: props.showOk,// 是否绘制确认按钮，且时分秒选择面板存在时绘制
              renderFooter: props.renderFooter,// ()=>{}绘制额外的尾部元素
              locale: locale,// 语言包
              prefixCls: prefixCls,// 样式类前缀
              showToday: props.showToday,// 是否绘制今天按钮
              disabledTime: disabledTime,// 影响时分秒选择面板
              showTimePicker: showTimePicker,// 显示时分秒选择面板标识
              showDateInput: props.showDateInput,// 显示浮层内输入框
              timePicker: timePicker,// 绘制时分秒选择面板节点
              selectedValue: selectedValue,// 选中的日期
              value: value,// 影响选中日期的时分秒
              disabledDate: disabledDate,// (moment)=>{}，判断时间是否可用
              okDisabled: !this.isAllowedDate(selectedValue),// 判断选中日期是否可用，影响确认按钮可点选状态
              onOk: this.onOk,// 确认按钮点击执行
              onSelect: this.onSelect,// (moment)=>{}，影响自定义renderFooter节点
              onToday: this.onToday,// 今天按钮点击执行
              onOpenTimePicker: this.openTimePicker,// 显示时分秒选择面板
              onCloseTimePicker: this.closeTimePicker// 隐藏时分秒选择面板
            })
          )
        )];
    
        return this.renderRoot({
          children: children,
          className: props.showWeekNumber ? prefixCls + '-week-number' : ''
        });
      }
    });
    
    // locale，语言包
    // prefixCls，样式类前缀
    // className，样式类
    // style，样式
    // visible，浮层日期选择面板是否显示
    // value，用于构成选中日期的时分秒，优先级高于state.defaultValue
    // defaultValue，用于构成选中日期的时分秒
    // selectedValue，用于构成选中日期的年月日，优先级高于state.defaultSelectedValue
    // defaultSelectedValue，用于构成选中日期的年月日
    // format，输入框格式化日期对象，优先级高于locale.dateTimeFormat或locale.dateFormat
    // renderSidebar，绘制自定义侧边栏
    // showToday，显示今天按钮
    // onClear，清空按钮点击时触发执行回调函数
    // showOk，显示确认按钮
    // onOk，确认按钮点击时触发执行回调函数
    // onKeyDown，键盘按键触发执行回调函数
    // onChange，时间选择面板、键盘按键时执行回调
    // onSelect，浮层内日期选择面板点击、今天按钮点击、清空按钮点击、浮层内输入框值改变、键盘enter键点击时执行回调
    // 
    // 影响timePicker时间选择面板的props属性：
    // timePicker，浮层内时间选择面板组件
    // 
    // 影响dateInput浮层内输入框的props属性：
    // showDateInput，是否显示输入框
    // dateInputPlaceholder，输入框的文案提示
    // 
    // 影响CalendarFooter浮层内尾部的props属性：
    // renderFooter，构成尾部除按钮外其他节点
    exports["default"] = Calendar;
    module.exports = exports['default'];
