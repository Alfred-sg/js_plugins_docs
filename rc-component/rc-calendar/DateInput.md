# DateInput

## 概述

绘制日期选择组件下，浮层内输入框组件。

## state属性设置

* str，输入框的值
* invalid，用户变更输入框的值后，其时间对象无效设为真，影响样式

## props属性设置

* prefixCls，样式类前缀
* placeholder，提示文本
* locale，语言包，用于设置清除按钮的文案
* format，将初始化props.selectedValue以format格式转化为字符串形式，"YYYY-MM-DD HH:mm:ss"形式
* disabled，浮层内输入框是否可用
* disabledDate=(moment)=>{}，判断moment对象是否可用
* selectedValue，输入框默认初始值moment对象
* onChange(moment|null)，浮层内输入框值改变时执行函数
* showClear，是否显示清除按钮
* onClear(null)，点击清除按钮时执行函数

## 源码

    'use strict';
    
    Object.defineProperty(exports, "__esModule", {
      value: true
    });
    
    var _react = require('react');
    var _react2 = _interopRequireDefault(_react);
    
    var _reactDom = require('react-dom');
    var _reactDom2 = _interopRequireDefault(_reactDom);
    
    var _moment = require('moment');
    var _moment2 = _interopRequireDefault(_moment);
    
    function _interopRequireDefault(obj) { return obj && obj.__esModule ? obj : { "default": obj }; }
    
    var DateInput = _react2["default"].createClass({
      displayName: 'DateInput',
    
      propTypes: {
        prefixCls: _react.PropTypes.string,
        timePicker: _react.PropTypes.object,// 无用
        value: _react.PropTypes.object,// 无用
        disabledTime: _react.PropTypes.any,// 无用
        format: _react.PropTypes.string,
        locale: _react.PropTypes.object,
        disabledDate: _react.PropTypes.func,
        onChange: _react.PropTypes.func,
        onClear: _react.PropTypes.func,
        placeholder: _react.PropTypes.string,
        onSelect: _react.PropTypes.func,// 无用
        selectedValue: _react.PropTypes.object
      },
    
      getInitialState: function getInitialState() {
        var selectedValue = this.props.selectedValue;
        return {
          str: selectedValue && selectedValue.format(this.props.format) || '',
          invalid: false
        };
      },
      componentWillReceiveProps: function componentWillReceiveProps(nextProps) {
        var selectedValue = nextProps.selectedValue;
        this.setState({
          str: selectedValue && selectedValue.format(nextProps.format) || '',
          invalid: false
        });
      },
      // 浮层内输入框值改变时执行函数，并调用props.onChange方法
      onInputChange: function onInputChange(event) {
        var str = event.target.value;
        this.setState({
          str: str
        });
        var value = void 0;
        var _props = this.props,
            disabledDate = _props.disabledDate,
            format = _props.format,
            onChange = _props.onChange;
    
        if (str) {
          var parsed = (0, _moment2["default"])(str, format, true);// 将输入框的值转化为moment对象
          if (!parsed.isValid()) {
            this.setState({
              invalid: true
            });
            return;
          }
          value = this.props.value.clone();
          value.year(parsed.year()).month(parsed.month()).date(parsed.date()).hour(parsed.hour())
            .minute(parsed.minute()).second(parsed.second());
    
          if (value && (!disabledDate || !disabledDate(value))) {
            var originalValue = this.props.selectedValue;
            if (originalValue && value) {
              if (!originalValue.isSame(value)) {
                onChange(value);
              }
            } else if (originalValue !== value) {
              onChange(value);
            }
          } else {
            this.setState({
              invalid: true
            });
            return;
          }
        } else {
          onChange(null);
        }
        this.setState({
          invalid: false
        });
      },
      // 清除按钮点击时执行函数，并调用props.onClear方法
      onClear: function onClear() {
        this.setState({
          str: ''
        });
        this.props.onClear(null);
      },
      // 获取当前节点
      getRootDOMNode: function getRootDOMNode() {
        return _reactDom2["default"].findDOMNode(this);
      },
      // 使输入框获得焦点
      focus: function focus() {
        this.refs.dateInput.focus();
      },
      render: function render() {
        var props = this.props;
        var _state = this.state,
            invalid = _state.invalid,
            str = _state.str;
        var locale = props.locale,
            prefixCls = props.prefixCls,
            placeholder = props.placeholder;
    
        var invalidClass = invalid ? prefixCls + '-input-invalid' : '';
        return _react2["default"].createElement(
          'div',
          { className: prefixCls + '-input-wrap' },
          _react2["default"].createElement(
            'div',
            { className: prefixCls + '-date-input-wrap' },
            _react2["default"].createElement('input', {
              ref: 'dateInput',
              className: prefixCls + '-input  ' + invalidClass,
              value: str,
              disabled: props.disabled,
              placeholder: placeholder,
              onChange: this.onInputChange
            })
          ),
          props.showClear ? _react2["default"].createElement('a', {
            className: prefixCls + '-clear-btn',
            role: 'button',
            title: locale.clear,
            onClick: this.onClear
          }) : null
        );
      }
    });
    
    // state：
    // str，输入框的值
    // invalid，用户变更输入框的值后，其时间对象无效设为真，影响样式
    
    // props：
    // prefixCls，样式类前缀
    // placeholder，提示文本
    // locale，语言包，用于设置清除按钮的文案
    // format，将初始化props.selectedValue以format格式转化为字符串形式，"YYYY-MM-DD HH:mm:ss"形式
    // disabled，浮层内输入框是否可用
    // disabledDate=(moment)=>{}，判断moment对象是否可用
    // selectedValue，输入框默认初始值moment对象
    // onChange(moment|null)，浮层内输入框值改变时执行函数
    // showClear，是否显示清除按钮
    // onClear(null)，点击清除按钮时执行函数
    exports["default"] = DateInput;
    module.exports = exports['default'];
