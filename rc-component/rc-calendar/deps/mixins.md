# CommonMixin、CalendarMixin

CommonMixin为日期选择组件注入shouldComponentUpdate、focus、getFormat方法；CalendarMixin为日期选择组件注入isAllowedDate、setValue、setSelectedValue、renderRoot、onSelect、componentWillReceiveProps方法

## CommonMixin

### 概述

为日期选择组件注入shouldComponentUpdate、focus、getFormat方法，设定特定的props。

### 源码

    'use strict';
    
    Object.defineProperty(exports, "__esModule", {
      value: true
    });
    
    var _react = require('react');
    
    var _en_US = require('../locale/en_US');
    var _en_US2 = _interopRequireDefault(_en_US);
    
    function _interopRequireDefault(obj) { return obj && obj.__esModule ? obj : { "default": obj }; }
    
    function noop() {}
    
    exports["default"] = {
      propTypes: {
        className: _react.PropTypes.string,
        locale: _react.PropTypes.object,
        style: _react.PropTypes.object,
        visible: _react.PropTypes.bool,
        onSelect: _react.PropTypes.func,
        prefixCls: _react.PropTypes.string,
        onChange: _react.PropTypes.func,
        onOk: _react.PropTypes.func
      },
    
      getDefaultProps: function getDefaultProps() {
        return {
          locale: _en_US2["default"],
          style: {},
          visible: true,
          prefixCls: 'rc-calendar',
          className: '',
          onSelect: noop,
          onChange: noop,
          onClear: noop,
          renderFooter: function renderFooter() {
            return null;
          },
          renderSidebar: function renderSidebar() {
            return null;
          }
        };
      },
      shouldComponentUpdate: function shouldComponentUpdate(nextProps) {
        return this.props.visible || nextProps.visible;
      },
      
      // 输入框格式化日期对象方式
      getFormat: function getFormat() {
        var format = this.props.format;
        var _props = this.props,
            locale = _props.locale,
            timePicker = _props.timePicker;
    
        if (!format) {
          if (timePicker) {
            format = locale.dateTimeFormat;
          } else {
            format = locale.dateFormat;
          }
        }
        return format;
      },
      focus: function focus() {
        if (this.refs.root) {
          this.refs.root.focus();
        }
      }
    };
    module.exports = exports['default'];



## CalendarMixin

### 概述

为日期选择组件注入isAllowedDate、setValue、setSelectedValue、renderRoot、onSelect、componentWillReceiveProps方法，设定特定的props、state。

### 源码

    'use strict';
    
    Object.defineProperty(exports, "__esModule", {
      value: true
    });
    
    var _defineProperty2 = require('babel-runtime/helpers/defineProperty');
    var _defineProperty3 = _interopRequireDefault(_defineProperty2);
    
    var _react = require('react');
    var _react2 = _interopRequireDefault(_react);
    
    var _classnames = require('classnames');
    var _classnames2 = _interopRequireDefault(_classnames);
    
    var _moment = require('moment');
    var _moment2 = _interopRequireDefault(_moment);
    
    var _index = require('../util/index');
    
    function _interopRequireDefault(obj) { return obj && obj.__esModule ? obj : { "default": obj }; }
    
    function noop() {}
    
    function getNow() {
      return (0, _moment2["default"])();
    }
    
    function getNowByCurrentStateValue(value) {
      var ret = void 0;
      if (value) {
        ret = (0, _index.getTodayTime)(value);
      } else {
        ret = getNow();
      }
      return ret;
    }
    
    var CalendarMixin = {
      propTypes: {
        value: _react.PropTypes.object,
        defaultValue: _react.PropTypes.object,
        onKeyDown: _react.PropTypes.func
      },
    
      getDefaultProps: function getDefaultProps() {
        return {
          onKeyDown: noop
        };
      },
      getInitialState: function getInitialState() {
        var props = this.props;
        var value = props.value || props.defaultValue || getNow();
        return {
          value: value,
          selectedValue: props.selectedValue || props.defaultSelectedValue
        };
      },
      componentWillReceiveProps: function componentWillReceiveProps(nextProps) {
        var value = nextProps.value;
        var selectedValue = nextProps.selectedValue;
    
        if ('value' in nextProps) {
          value = value || nextProps.defaultValue || getNowByCurrentStateValue(this.state.value);
          this.setState({
            value: value
          });
        }
        if ('selectedValue' in nextProps) {
          this.setState({
            selectedValue: selectedValue
          });
        }
      },
      // 浮层内日期选择面板点击、今天按钮点击、清空按钮点击、浮层内输入框值改变、键盘enter键点击时执行回调
      onSelect: function onSelect(value, cause) {
        if (value) {
          this.setValue(value);
        }
        this.setSelectedValue(value, cause);
      },
      // 绘制根节点及浮层面板
      renderRoot: function renderRoot(newProps) {
        var _className;
    
        var props = this.props;
        var prefixCls = props.prefixCls;
    
        var className = (_className = {}, (0, _defineProperty3["default"])(_className, prefixCls, 1), (0, _defineProperty3["default"])(_className, prefixCls + '-hidden', !props.visible), (0, _defineProperty3["default"])(_className, props.className, !!props.className), (0, _defineProperty3["default"])(_className, newProps.className, !!newProps.className), _className);
    
        return _react2["default"].createElement(
          'div',
          {
            ref: 'root',
            className: '' + (0, _classnames2["default"])(className),
            style: this.props.style,
            tabIndex: '0',
            onKeyDown: this.onKeyDown
          },
          newProps.children
        );
      },
      setSelectedValue: function setSelectedValue(selectedValue, cause) {
        // if (this.isAllowedDate(selectedValue)) {
        if (!('selectedValue' in this.props)) {
          this.setState({
            selectedValue: selectedValue
          });
        }
        this.props.onSelect(selectedValue, cause);
        // }
      },
      setValue: function setValue(value) {
        var originalValue = this.state.value;
        if (!('value' in this.props)) {
          this.setState({
            value: value
          });
        }
        if (originalValue && value && !originalValue.isSame(value) || !originalValue && value || originalValue && !value) {
          this.props.onChange(value);
        }
      },
      isAllowedDate: function isAllowedDate(value) {
        var disabledDate = this.props.disabledDate;
        var disabledTime = this.props.disabledTime;
        return (0, _index.isAllowedDate)(value, disabledDate, disabledTime);
      }
    };
    
    exports["default"] = CalendarMixin;
    module.exports = exports['default'];