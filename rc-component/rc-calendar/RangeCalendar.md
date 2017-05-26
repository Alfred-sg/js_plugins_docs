# RangeCalendar、CalendarPart

RangeCalendar绘制日期范围选择组件，CalendarPart选择日选择组件。

## RangeCalendar

### 概述

绘制日期范围选择组件。

### state属性

* selectedValue，日期选择组件中，以日份点选组件或输入框改变this.state.selectedValue获取时分秒值
* hoverValue，获得焦点的前后日期，面板获得焦点时赋值为this.state.selectedValue
* value，日期选择组件中，以年月点选组件改变this.state.value获取年月日值
* showTimePicker，显示时间选择面板标识符

### props属性

* visible，显示日期范围选择器标识符
* showToday，是否显示今天按钮
* type，浮层内日期选择组件以何种形式设置前置日期、后置日期
* renderSidebar，自定义设置侧边栏等
* prefixCls，样式类前缀
* className，样式类
* style，样式
* showWeekNumber，是否显示星期数
* selectedValue，选中的日期，优先级高于props.defaultSelectedValue
* defaultSelectedValue，选中的日期
* value，作为初始化this.state.value，影响年月，优先级高于props.defaultValue
* defaultValue，作为初始化this.state.value，影响年月
* showOk，是否显示确认按钮
* onOk=([moment,moment])=>{}，点击确认按钮回调
* onSelect([moment,moment])，清空按钮点击、或前置后置日期选中时触发执行函数
* onChange([moment,moment])，清空按钮点击、日期面板选中、或浮层输入框值改变时触发执行函数
* onValueChange(moment)，浮层内头部年、月点选节点改变值时执行函数，年份、月份选择面板点选时执行

* 影响清空按钮的props属性：
* showClear，是否显示清空按钮
* locale，语言包，为浮层内日期选择组件中清除按钮设置文案、日期选择组件年日选择头、今天按钮文案等
* onClear，点击清除按钮时执行函数

* 影响CalendarPart组件浮层内容的props属性：
* direction，正向或反向显示前后日期，"left"日期范围选择器左面板显示前一个日期、右面板显示后一个日期
* format，浮层内输入框格式化时间对象的形式
* locale，语言包，为浮层内日期选择组件中清除按钮设置文案、日期选择组件年日选择头、今天按钮文案等
* prefixCls，样式类前缀
* disabledDate=(moment,props.value)=>{}，判断moment对象是否可用
* dateInputPlaceholder，浮层内日期选择组件输入框的输入提示文本
* selectedValue，初始化选中的日期
* value，选中的日期、以及获取moment对象的语言包、以显示星期名
* onValueChange(moment)，浮层内头部年、月点选节点改变值时执行函数，年份、月份选择面板点选时执行
* timePicker，以节点形式作为时分秒组件渲染，同时其props属性设置时分选择组件的props
* disabledTime=(moment,["start"|"end"])=>{}，返回值作为时分选择组件的props.defaultDisabledTime属性
* timePickerDisabledTime，设置时分选择组件的props
* hoverValue，浮层内日期组件点选后的日期
* dateRender=(moment,props.value)，用于绘制浮层内日期渲染组件内容
* onSelect(moment)，浮层内日期选择组件点选时执行函数
* onDayHover(moment)，浮层内日期选择组件鼠标移入时执行函数
* showWeekNumber，是否显示日期组件中的日期数

* 影响TodayButton今天按钮的props属性：
* prefixCls，样式类前缀
* locale，语言包，为浮层内日期选择组件中清除按钮设置文案、日期选择组件年日选择头、今天按钮文案等
* timePicker，以节点形式作为时分秒组件渲染，同时其props属性设置时分选择组件的props
* disabledDate(moment,props.value)，判断moment日期节点是否不可点选

* 影响TimePickerButton日期、时间面板切换按钮的props属性：
* prefixCls，样式类前缀
* locale，语言包，日期、时间面板切换按钮文案

* 影响OkButton确认按钮的props属性：
* prefixCls，样式类前缀
* locale，语言包，确认按钮文案

### 源码

    'use strict';
    
    Object.defineProperty(exports, "__esModule", {
      value: true
    });
    
    var _extends2 = require('babel-runtime/helpers/extends');
    var _extends3 = _interopRequireDefault(_extends2);
    
    var _slicedToArray2 = require('babel-runtime/helpers/slicedToArray');
    var _slicedToArray3 = _interopRequireDefault(_slicedToArray2);
    
    var _defineProperty2 = require('babel-runtime/helpers/defineProperty');
    var _defineProperty3 = _interopRequireDefault(_defineProperty2);
    
    var _typeof2 = require('babel-runtime/helpers/typeof');
    var _typeof3 = _interopRequireDefault(_typeof2);
    
    var _react = require('react');
    var _react2 = _interopRequireDefault(_react);
    
    var _moment = require('moment');
    var _moment2 = _interopRequireDefault(_moment);
    
    var _classnames2 = require('classnames');
    var _classnames3 = _interopRequireDefault(_classnames2);
    
    var _CalendarPart = require('./range-calendar/CalendarPart');
    var _CalendarPart2 = _interopRequireDefault(_CalendarPart);
    
    var _util = require('./util/');
    
    var _TodayButton = require('./calendar/TodayButton');
    var _TodayButton2 = _interopRequireDefault(_TodayButton);
    
    var _OkButton = require('./calendar/OkButton');
    var _OkButton2 = _interopRequireDefault(_OkButton);
    
    var _TimePickerButton = require('./calendar/TimePickerButton');
    var _TimePickerButton2 = _interopRequireDefault(_TimePickerButton);
    
    var _CommonMixin = require('./mixin/CommonMixin');
    var _CommonMixin2 = _interopRequireDefault(_CommonMixin);
    
    function _interopRequireDefault(obj) { return obj && obj.__esModule ? obj : { "default": obj }; }
    
    function noop() {}
    
    function getNow() {
      return (0, _moment2["default"])();
    }
    
    // 浮层内头部年、月点选节点改变值时执行函数，年份、月份选择面板点选时执行函数
    function onValueChange(direction, current) {
      var value = void 0;
      value = current;
      if (direction === 'right') {
        value.add(-1, 'months');
      }
      this.fireValueChange(value);
    }
    
    // 参数init为真取defaultSelectedValue、defaultValue，为0时不取defaultSelectedValue、defaultValue
    function normalizeAnchor(props, init) {
      var selectedValue = props.selectedValue || init && props.defaultSelectedValue || [];
      var value = props.value;
      if (Array.isArray(value)) {
        value = value[0];
      }
      var defaultValue = props.defaultValue;
      if (Array.isArray(defaultValue)) {
        defaultValue = defaultValue[0];
      }
      return value || init && defaultValue || selectedValue[0] || init && getNow();
    }
    
    // 生成从0到length的数组
    function generateOptions(length) {
      var arr = [];
      for (var value = 0; value < length; value++) {
        arr.push(value);
      }
      return arr;
    }
    
    // 浮层内输入框值改变时，按direction将输入值作校验后赋值给state.selectedValue
    // 浮层内左右两侧输入框均有值、且右值大于左值时才起作用
    function onInputSelect(direction, value) {
      if (!value) {
        return;
      }
      var originalValue = this.state.selectedValue;
      var selectedValue = originalValue.concat();
      var index = direction === 'left' ? 0 : 1;
      selectedValue[index] = value;
      if (selectedValue[0] && this.compare(selectedValue[0], selectedValue[1]) > 0) {
        selectedValue[1 - index] = this.state.showTimePicker ? selectedValue[index] : undefined;
      }
      this.fireSelectValueChange(selectedValue);
    }
    
    var RangeCalendar = _react2["default"].createClass({
      displayName: 'RangeCalendar',
    
      propTypes: {
        prefixCls: _react.PropTypes.string,
        dateInputPlaceholder: _react.PropTypes.any,
        defaultValue: _react.PropTypes.any,
        timePicker: _react.PropTypes.any,
        value: _react.PropTypes.any,
        showOk: _react.PropTypes.bool,
        showToday: _react.PropTypes.bool,
        selectedValue: _react.PropTypes.array,
        defaultSelectedValue: _react.PropTypes.array,
        onOk: _react.PropTypes.func,
        showClear: _react.PropTypes.bool,
        locale: _react.PropTypes.object,
        onChange: _react.PropTypes.func,
        onSelect: _react.PropTypes.func,
        onValueChange: _react.PropTypes.func,
        disabledDate: _react.PropTypes.func,
        format: _react.PropTypes.oneOfType([_react.PropTypes.object, _react.PropTypes.string]),
        onClear: _react.PropTypes.func,
        type: _react.PropTypes.any,
        disabledTime: _react.PropTypes.func
      },
    
      mixins: [_CommonMixin2["default"]],
    
      getDefaultProps: function getDefaultProps() {
        return {
          type: 'both',
          defaultSelectedValue: [],
          onValueChange: noop,
          disabledTime: noop,
          showToday: true
        };
      },
      getInitialState: function getInitialState() {
        var props = this.props;
        var selectedValue = props.selectedValue || props.defaultSelectedValue;
        var value = normalizeAnchor(props, 1);
        return {
          selectedValue: selectedValue,// 日期选择组件中，以日份点选组件或输入框改变this.state.selectedValue获取日值
          hoverValue: [],// 获得焦点的前后日期，面板获得焦点时赋值为this.state.selectedValue
          value: value,// 日期选择组件中，以年月点选组件改变this.state.value获取年月值
          showTimePicker: false// 显示时间选择面板标识符
        };
      },
      componentWillReceiveProps: function componentWillReceiveProps(nextProps) {
        var newState = {};
        if ('value' in nextProps) {
          if (nextProps.value) {
            newState.value = nextProps.value;
          } else {
            newState.value = normalizeAnchor(nextProps, 0);
          }
          this.setState(newState);
        }
        if ('selectedValue' in nextProps) {
          newState.selectedValue = nextProps.selectedValue;
          this.setState(newState);
        }
      },
    
      // 鼠标移入、移除日期选择面板时执行函数
      onDatePanelEnter: function onDatePanelEnter() {
        if (this.hasSelectedValue()) {
          this.setState({
            hoverValue: this.state.selectedValue.concat()
          });
        }
      },
      onDatePanelLeave: function onDatePanelLeave() {
        if (this.hasSelectedValue()) {
          this.setState({
            hoverValue: []
          });
        }
      },
    
      // (moment)=>{}，日期选择组件点击时执行函数，按props.type属性设置选中的前置日期、后置日期
      onSelect: function onSelect(value) {
        var _state = this.state,
            hoverValue = _state.hoverValue,
            selectedValue = _state.selectedValue;
    
        var nextSelectedValue = void 0;
        var type = this.props.type;
    
        var changed = false;
    
        // type="both"时，日期选择组件点选时重新选择日期
        if (!hoverValue[0] && !hoverValue[1] && type === 'both') {
          nextSelectedValue = [value];
          changed = true;
    
        // type="start"时，日期选择组件若只选中一个、或选中的后置日期小于当前点选值，则重新选择日期
        // 若已选中后置日期、且选中的后置日期大于当前点选值，本次点选作为前置日期
        } else if (type === 'start') {
          var endValue = selectedValue[1];
          if (!endValue || this.compare(endValue, value) < 0) {
            nextSelectedValue = [value];
          } else {
            nextSelectedValue = [value, endValue];
          }
          changed = true;
    
        // type="end"时，日期选择组件若已选中前置日期、且选中的前置日期小于当前点选值，本次点选作为后置日期
        //    若未选中前置日期、或选中的前置日期大于当前点选值，则重新选择日期
        // type为其他值时，日期选择组件若已有前置焦点日期、且前置焦点日期小于当前点选值，本次点选作为后置日期，前置焦点日期作为前置日期
        //    若未曾有前置焦点日期、或前置焦点日期大于当前点选值，则以当前选中值重新选择日期
        } else {
          var startValue = void 0;
          startValue = type === 'end' ? selectedValue[0] : hoverValue[0];
          if (startValue && this.compare(startValue, value) <= 0) {
            nextSelectedValue = [startValue, value];
            changed = true;
          } else {
            nextSelectedValue = [value];
            changed = true;
          }
        }
    
        if (changed) {
          this.fireSelectValueChange(nextSelectedValue);
        }
      },
    
      // (moment)=>{}，浮层内日期选择组件鼠标移入时执行函数，更新this.state.hoverValue
      onDayHover: function onDayHover(value) {
        var hoverValue = this.state.hoverValue;
        var selectedValue = this.state.selectedValue;
        var type = this.props.type;
    
        // 若当前鼠标移入的日期小于选中的后置节点，当前鼠标移入的焦点作为前置鼠标移入的日期
        if (type === 'start' && selectedValue[1]) {
          if (this.compare(value, selectedValue[1]) < 0) {
            hoverValue = [value, selectedValue[1]];
          } else {
            hoverValue = [value];
          }
          this.setState({
            hoverValue: hoverValue
          });
        } else if (type === 'end' && selectedValue[0]) {
          if (this.compare(value, selectedValue[0]) > 0) {
            hoverValue = [selectedValue[0], value];
          } else {
            hoverValue = [];
          }
          this.setState({
            hoverValue: hoverValue
          });
        } else {
          if (!hoverValue[0] || this.compare(value, hoverValue[0]) < 0) {
            return;
          }
          hoverValue[1] = value;
          this.setState({
            hoverValue: hoverValue
          });
        }
      },
    
      // 今天按钮点击时触发执行，变更this.state.value
      onToday: function onToday() {
        this.setState({
          value: (0, _util.getTodayTime)(this.state.value)
        });
      },
    
      // 显示时间选择面板时执行函数
      onOpenTimePicker: function onOpenTimePicker() {
        this.setState({
          showTimePicker: true
        });
      },
    
      // 隐藏时间选择面板时执行函数
      onCloseTimePicker: function onCloseTimePicker() {
        this.setState({
          showTimePicker: false
        });
      },
    
      // 点击确认按钮执行函数
      onOk: function onOk() {
        var selectedValue = this.state.selectedValue;
    
        if (this.isAllowedDateAndTime(selectedValue)) {
          this.props.onOk(this.state.selectedValue);
        }
      },
    
      // (moment)=>{}，浮层内输入框值改变时执行函数
      onStartInputSelect: function onStartInputSelect() {
        for (var _len = arguments.length, oargs = Array(_len), _key = 0; _key < _len; _key++) {
          oargs[_key] = arguments[_key];
        }
    
        var args = ['left'].concat(oargs);
        return onInputSelect.apply(this, args);
      },
      onEndInputSelect: function onEndInputSelect() {
        for (var _len2 = arguments.length, oargs = Array(_len2), _key2 = 0; _key2 < _len2; _key2++) {
          oargs[_key2] = arguments[_key2];
        }
    
        var args = ['right'].concat(oargs);
        return onInputSelect.apply(this, args);
      },
    
      // (moment)=>{}，浮层内头部年、月点选节点改变值时执行函数，年份、月份选择面板点选时执行函数
      onStartValueChange: function onStartValueChange() {
        for (var _len3 = arguments.length, oargs = Array(_len3), _key3 = 0; _key3 < _len3; _key3++) {
          oargs[_key3] = arguments[_key3];
        }
    
        var args = ['left'].concat(oargs);
        return onValueChange.apply(this, args);
      },
      onEndValueChange: function onEndValueChange() {
        for (var _len4 = arguments.length, oargs = Array(_len4), _key4 = 0; _key4 < _len4; _key4++) {
          oargs[_key4] = arguments[_key4];
        }
    
        var args = ['right'].concat(oargs);
        return onValueChange.apply(this, args);
      },
    
      // 取this.state.selectedValue的年月日，this.state.value的时分秒，构成选中的起始日期
      getStartValue: function getStartValue() {
        var value = this.state.value;// 取value的时分秒，再取state.selectedValue的年月日，构成选中的日期对象
        var selectedValue = this.state.selectedValue;
        if (selectedValue[0] && this.props.timePicker) {
          value = value.clone();
          (0, _util.syncTime)(selectedValue[0], value);
        }
        return value;
      },
      getEndValue: function getEndValue() {
        var endValue = this.state.value.clone();
        endValue.add(1, 'months');
        var selectedValue = this.state.selectedValue;
    
        if (selectedValue[1] && this.props.timePicker) {
          (0, _util.syncTime)(selectedValue[1], endValue);
        }
        if (this.state.showTimePicker) {
          return selectedValue[1] ? selectedValue[1] : this.getStartValue();
        }
        return endValue;
      },
    
      // 切换到时分秒选择面板timePicker时，若选中的前置日期、后置日期的年月日相等，后置时分秒不可超前于前置时分秒
      getEndDisableTime: function getEndDisableTime() {
        var _state2 = this.state,
            selectedValue = _state2.selectedValue,
            value = _state2.value;
    
        var startValue = selectedValue && selectedValue[0] || value.clone();
    
        if (!selectedValue[1] || startValue.isSame(selectedValue[1], 'day')) {
          var _ret = function () {
            var hours = startValue.hour();
            var minutes = startValue.minute();
            var second = startValue.second();
            var _disabledHours = generateOptions(hours);
            var _disabledMinutes = generateOptions(minutes);
            var _disabledSeconds = generateOptions(second);
            return {
              v: {
                disabledHours: function disabledHours() {
                  return _disabledHours;
                },
                disabledMinutes: function disabledMinutes(hour) {
                  if (hour === hours) {
                    return _disabledMinutes;
                  }
                  return [];
                },
                disabledSeconds: function disabledSeconds(hour, minute) {
                  if (hour === hours && minute === minutes) {
                    return _disabledSeconds;
                  }
                  return [];
                }
              }
            };
          }();
    
          if ((typeof _ret === 'undefined' ? 'undefined' : (0, _typeof3["default"])(_ret)) === "object") 
            return _ret.v;
        }
        return null;
      },
    
      // 以props.disabledDate、props.disabledTime校验selectedValue是否有效
      isAllowedDateAndTime: function isAllowedDateAndTime(selectedValue) {
        return (0, _util.isAllowedDate)(selectedValue[0], this.props.disabledDate, this.disabledStartTime) && 
          (0, _util.isAllowedDate)(selectedValue[1], this.props.disabledDate, this.disabledEndTime);
      },
    
      // 日期范围选择器是否选中前置、后置日期
      hasSelectedValue: function hasSelectedValue() {
        var selectedValue = this.state.selectedValue;
      
        return !!selectedValue[1] && !!selectedValue[0];
      },
    
      // 比较两个moment对象v1、v2；this.props.timePicker为真时以秒作比较，否则以日作比较
      compare: function compare(v1, v2) {
        if (this.props.timePicker) {
          return v1.diff(v2);
        }
        return v1.diff(v2, 'days');
      },
    
      // 浮层内日期组件点选、输入框值变更、或点击清空按钮时执行函数；点击清空按钮时，参数direct为真
      // 输入框值变更时，须左右两侧输入框均有值、且右值大于左值时才起执行this.fireSelectValueChange方法
      fireSelectValueChange: function fireSelectValueChange(selectedValue, direct) {
        if (!('selectedValue' in this.props)) {
          this.setState({
            selectedValue: selectedValue
          });
        }
    
        // 日期选择面板仅选中单个值，更改this.state.value
        if (!this.state.selectedValue[0] || !this.state.selectedValue[1]) {
          this.setState({
            selectedValue: selectedValue,
            value: selectedValue[0] || getNow()
          });
        }
    
        // 仅前置日期选中时，this.state.hoverValue赋值为this.state.selectedValue
        if (selectedValue[0] && !selectedValue[1]) {
          this.setState({
            hoverValue: selectedValue.concat()
          });
        }
    
        this.props.onChange(selectedValue);
    
        // 清空按钮点击时，或前置、后置日期点选时触发执行条件分支
        if (direct || selectedValue[0] && selectedValue[1]) {
          this.setState({
            hoverValue: []
          });
          this.props.onSelect(selectedValue);
        }
      },
    
      // 浮层内头部年、月点选节点改变值时执行函数，年份、月份选择面板点选时执行函数
      fireValueChange: function fireValueChange(value) {
        var props = this.props;
        if (!('value' in props)) {
          this.setState({
            value: value
          });
        }
        props.onValueChange(value);
      },
    
      // 点击清空按钮时执行函数，并调用props.onClear方法
      clear: function clear() {
        this.fireSelectValueChange([], true);
        this.props.onClear();
      },
    
      // 返回值构成时分选择组件的props.defaultDisabledTime属性
      disabledStartTime: function disabledStartTime(time) {
        return this.props.disabledTime(time, 'start');
      },
      disabledEndTime: function disabledEndTime(time) {
        return this.props.disabledTime(time, 'end');
      },
    
      render: function render() {
        var _className, _classnames;
    
        var props = this.props;
        var state = this.state;
        var showTimePicker = state.showTimePicker;
        var prefixCls = props.prefixCls,
            dateInputPlaceholder = props.dateInputPlaceholder,
            timePicker = props.timePicker,
            showOk = props.showOk,
            locale = props.locale,
            showClear = props.showClear,
            showToday = props.showToday,
            type = props.type;
        var hoverValue = state.hoverValue,
            selectedValue = state.selectedValue;
    
        // 日期范围选择器样式类
        var className = (
          _className = {}, 
          (0, _defineProperty3["default"])(_className, props.className, !!props.className), 
          (0, _defineProperty3["default"])(_className, prefixCls, 1), 
          (0, _defineProperty3["default"])(_className, prefixCls + '-hidden', !props.visible), 
          (0, _defineProperty3["default"])(_className, prefixCls + '-range', 1), 
          (0, _defineProperty3["default"])(_className, prefixCls + '-show-time-picker', showTimePicker), 
          (0, _defineProperty3["default"])(_className, prefixCls + '-week-number', props.showWeekNumber), 
          _className
        );
        var classes = (0, _classnames3["default"])(className);
    
        var newProps = {
          selectedValue: state.selectedValue,
          // (moment)=>{}，浮层内日期选择组件点选时执行函数
          onSelect: this.onSelect,
          // (moment)=>{}，浮层内日期选择组件鼠标移入时执行函数
          onDayHover: type === 'start' && selectedValue[1] || type === 'end' && selectedValue[0] || 
            !!hoverValue.length ? this.onDayHover : undefined
        };
    
        // 输入框提示文案
        var placeholder1 = void 0;
        var placeholder2 = void 0;
        if (dateInputPlaceholder) {
          if (Array.isArray(dateInputPlaceholder)) {
            var _dateInputPlaceholder = (0, _slicedToArray3["default"])(dateInputPlaceholder, 2);
    
            placeholder1 = _dateInputPlaceholder[0];
            placeholder2 = _dateInputPlaceholder[1];
          } else {
            placeholder1 = placeholder2 = dateInputPlaceholder;
          }
        }
    
        var showOkButton = showOk === true || showOk !== false && !!timePicker;
    
        // 按钮组样式类
        var cls = (0, _classnames3["default"])((
          _classnames = {}, 
          (0, _defineProperty3["default"])(_classnames, prefixCls + '-footer', true), 
          (0, _defineProperty3["default"])(_classnames, prefixCls + '-range-bottom', true), 
          (0, _defineProperty3["default"])(_classnames, prefixCls + '-footer-show-ok', showOkButton), 
          _classnames
        ));
    
        var startValue = this.getStartValue();
        var endValue = this.getEndValue();
    
        var todayTime = (0, _util.getTodayTime)(startValue);
        var thisMonth = todayTime.month();
        var thisYear = todayTime.year();
        var isThisYear = thisYear === startValue.year() || this.year === endValue.year();
        var isTodayInView = isThisYear && (thisMonth === startValue.month() || thisMonth === endValue.month());
    
        return _react2["default"].createElement(
          'div',
          {
            ref: 'root',
            className: classes,
            style: props.style,
            tabIndex: '0'
          },
          
          props.renderSidebar(),
    
          _react2["default"].createElement(
            'div',
            { className: prefixCls + '-panel' },
    
            // 绘制清空按钮
            showClear && selectedValue[0] && selectedValue[1] ? _react2["default"].createElement('a', {
              className: prefixCls + '-clear-btn',
              role: 'button',
              title: locale.clear,
              onClick: this.clear
            }) : null,
    
            _react2["default"].createElement(
              'div',
              {
                className: prefixCls + '-date-panel',
                onMouseLeave: type !== 'both' ? this.onDatePanelLeave : undefined,
                onMouseEnter: type !== 'both' ? this.onDatePanelEnter : undefined
              },
    
              // 浮层内日期选择组件
              _react2["default"].createElement(_CalendarPart2["default"], (0, _extends3["default"])({}, props, newProps, {
                hoverValue: hoverValue,// 浮层内日期组件点选后的日期
                direction: 'left',// 正向或反向显示前后日期，"left"日期范围选择器左面板显示前一个日期、右面板显示后一个日期
                disabledTime: this.disabledStartTime,// (value)=>{}，返回值作为时分选择组件的props.defaultDisabledTime属性
                format: this.getFormat(),// 浮层内输入框格式化时间对象的形式
                value: startValue,// 选中的日期、以及获取moment对象的语言包、以显示星期名
                placeholder: placeholder1,// 浮层内输入框提示文案
                onInputSelect: this.onStartInputSelect,// (moment)=>{}，浮层内输入框值改变时执行函数
                onValueChange: this.onStartValueChange,// (moment)=>{}，浮层内头部年、月点选节点改变值时执行函数
                timePicker: timePicker,// 以节点形式作为时分秒组件渲染，同时其props属性设置时分选择组件的props
                showTimePicker: showTimePicker// 是否显示时分秒选择组件
              })),
              _react2["default"].createElement(
                'span',
                { className: prefixCls + '-range-middle' },
                '~'
              ),
              _react2["default"].createElement(_CalendarPart2["default"], (0, _extends3["default"])({}, props, newProps, {
                hoverValue: hoverValue,
                direction: 'right',
                format: this.getFormat(),
                timePickerDisabledTime: this.getEndDisableTime(),
                placeholder: placeholder2,
                value: endValue,
                onInputSelect: this.onEndInputSelect,
                onValueChange: this.onEndValueChange,
                timePicker: timePicker,
                showTimePicker: showTimePicker,
                disabledTime: this.disabledEndTime
              }))
            ),
    
            // 今天按钮、切换时间选择面板按钮、确认按钮
            _react2["default"].createElement(
              'div',
              { className: cls },
              props.renderFooter(),
              showToday || props.timePicker || showOkButton ? _react2["default"].createElement(
                'div',
                { className: prefixCls + '-footer-btn' },
    
                // 今天按钮
                showToday ? _react2["default"].createElement(_TodayButton2["default"], (0, _extends3["default"])({}, props, {
                  disabled: isTodayInView,// 今天按钮是否可点击
                  value: state.value,// 选中的日期对象
                  onToday: this.onToday,// 今天按钮点击时执行函数
                  text: locale.backToToday// 自定义今天按钮文案，由props.locale语言包决定
                })) : null,
    
                // 日期、时间面板切换按钮
                props.timePicker ? _react2["default"].createElement(_TimePickerButton2["default"], (0, _extends3["default"])({}, props, {
                  showTimePicker: showTimePicker,// 时间选择面板显示中状态
                  onOpenTimePicker: this.onOpenTimePicker,// 显示时间选择面板时执行函数
                  onCloseTimePicker: this.onCloseTimePicker,// 隐藏时间选择面板时执行函数
                  timePickerDisabled: !this.hasSelectedValue() || hoverValue.length// 日期、时间面板切换按钮是否可用
                })) : null,
    
                // 确认按钮
                showOkButton ? _react2["default"].createElement(_OkButton2["default"], (0, _extends3["default"])({}, props, {
                  value: state.value,// 无用
                  onOk: this.onOk,// 确认按钮点击执行函数
                  okDisabled: !this.isAllowedDateAndTime(selectedValue) || !this.hasSelectedValue() || hoverValue.length// 确认按钮是否可用
                })) : null
              ) : null
            )
          )
        );
      }
    });
    
    // visible，显示日期范围选择器标识符
    // showToday，是否显示今天按钮
    // type，浮层内日期选择组件以何种形式设置前置日期、后置日期
    // renderSidebar，自定义设置侧边栏等
    // prefixCls，样式类前缀
    // className，样式类
    // style，样式
    // showWeekNumber，是否显示星期数
    // selectedValue，选中的日期，优先级高于props.defaultSelectedValue
    // defaultSelectedValue，选中的日期
    // value，作为初始化this.state.value，影响年月，优先级高于props.defaultValue
    // defaultValue，作为初始化this.state.value，影响年月
    // showOk，是否显示确认按钮
    // onOk=([moment,moment])=>{}，点击确认按钮回调
    // onSelect([moment,moment])，清空按钮点击、或前置后置日期选中时触发执行函数
    // onChange([moment,moment])，清空按钮点击、日期面板选中、或浮层输入框值改变时触发执行函数
    // onValueChange(moment)，浮层内头部年、月点选节点改变值时执行函数，年份、月份选择面板点选时执行
    // 
    // 影响清空按钮的props属性：
    // showClear，是否显示清空按钮
    // locale，语言包，为浮层内日期选择组件中清除按钮设置文案、日期选择组件年日选择头、今天按钮文案等
    // onClear，点击清除按钮时执行函数
    // 
    // 影响CalendarPart组件浮层内容的props属性：
    // direction，正向或反向显示前后日期，"left"日期范围选择器左面板显示前一个日期、右面板显示后一个日期
    // format，浮层内输入框格式化时间对象的形式
    // locale，语言包，为浮层内日期选择组件中清除按钮设置文案、日期选择组件年日选择头、今天按钮文案等
    // prefixCls，样式类前缀
    // disabledDate=(moment,props.value)=>{}，判断moment对象是否可用
    // dateInputPlaceholder，浮层内日期选择组件输入框的输入提示文本
    // selectedValue，初始化选中的日期
    // value，选中的日期、以及获取moment对象的语言包、以显示星期名
    // onValueChange(moment)，浮层内头部年、月点选节点改变值时执行函数，年份、月份选择面板点选时执行
    // timePicker，以节点形式作为时分秒组件渲染，同时其props属性设置时分选择组件的props
    // disabledTime=(moment,["start"|"end"])=>{}，返回值作为时分选择组件的props.defaultDisabledTime属性
    // timePickerDisabledTime，设置时分选择组件的props
    // hoverValue，浮层内日期组件点选后的日期
    // dateRender=(moment,props.value)，用于绘制浮层内日期渲染组件内容
    // onSelect(moment)，浮层内日期选择组件点选时执行函数
    // onDayHover(moment)，浮层内日期选择组件鼠标移入时执行函数
    // showWeekNumber，是否显示日期组件中的日期数
    // 
    // 影响TodayButton今天按钮的props属性：
    // prefixCls，样式类前缀
    // locale，语言包，为浮层内日期选择组件中清除按钮设置文案、日期选择组件年日选择头、今天按钮文案等
    // timePicker，以节点形式作为时分秒组件渲染，同时其props属性设置时分选择组件的props
    // disabledDate(moment,props.value)，判断moment日期节点是否不可点选
    // 
    // 影响TimePickerButton日期、时间面板切换按钮的props属性：
    // prefixCls，样式类前缀
    // locale，语言包，日期、时间面板切换按钮文案
    // 
    // 影响OkButton确认按钮的props属性：
    // prefixCls，样式类前缀
    // locale，语言包，确认按钮文案
    exports["default"] = RangeCalendar;
    module.exports = exports['default'];



## CalendarPart

### 概述

绘制日期范围组件中，浮层内容节点。

### props属性

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

### 源码

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
