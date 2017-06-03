# CalendarFooter

## 概述

绘制浮层面板底部节点。

## props属性

* prefixCls，样式类前缀
* showToday，是否绘制今天按钮
* showOk，是否绘制确认按钮，且时分秒选择面板存在时绘制
* timePicker，自定义时分秒选择面板存在时，绘制日期、时分秒面板切换按钮
* value，影响今天按钮的时间
* renderFooter，()=>{}绘制额外的尾部元素
* defaultValue，影响自定义renderFooter节点
* selectedValue，影响自定义renderFooter节点
* showDateInput，影响自定义renderFooter节点
* disabledTime，影响自定义renderFooter节点
* onSelect，(moment)=>{}，影响自定义renderFooter节点
* onOk，确认按钮点击执行
* onToday，今天按钮点击执行
* onOpenTimePicker，显示时分秒选择面板
* onCloseTimePicker，隐藏时分秒选择面板
* local，语言包

## 源码

    'use strict';
    
    Object.defineProperty(exports, "__esModule", {
      value: true
    });
    
    var _defineProperty2 = require('babel-runtime/helpers/defineProperty');
    var _defineProperty3 = _interopRequireDefault(_defineProperty2);
    
    var _extends2 = require('babel-runtime/helpers/extends');
    var _extends3 = _interopRequireDefault(_extends2);
    
    var _react = require('react');
    var _react2 = _interopRequireDefault(_react);
    
    var _reactDom = require('react-dom');
    var _reactDom2 = _interopRequireDefault(_reactDom);
    
    var _mapSelf = require('rc-util/lib/Children/mapSelf');
    var _mapSelf2 = _interopRequireDefault(_mapSelf);
    
    var _classnames = require('classnames');
    var _classnames2 = _interopRequireDefault(_classnames);
    
    var _TodayButton = require('../calendar/TodayButton');
    var _TodayButton2 = _interopRequireDefault(_TodayButton);
    
    var _OkButton = require('../calendar/OkButton');
    var _OkButton2 = _interopRequireDefault(_OkButton);
    
    var _TimePickerButton = require('../calendar/TimePickerButton');
    var _TimePickerButton2 = _interopRequireDefault(_TimePickerButton);
    
    function _interopRequireDefault(obj) { return obj && obj.__esModule ? obj : { "default": obj }; }
    
    var CalendarFooter = _react2["default"].createClass({
      displayName: 'CalendarFooter',
    
      propTypes: {
        prefixCls: _react.PropTypes.string,
        showDateInput: _react.PropTypes.bool,
        disabledTime: _react.PropTypes.any,
        timePicker: _react.PropTypes.element,
        selectedValue: _react.PropTypes.any,
        showOk: _react.PropTypes.bool,
        onSelect: _react.PropTypes.func,
        value: _react.PropTypes.object,
        renderFooter: _react.PropTypes.func,
        defaultValue: _react.PropTypes.object
      },
    
      onSelect: function onSelect(value) {
        this.props.onSelect(value);
      },
      getRootDOMNode: function getRootDOMNode() {
        return _reactDom2["default"].findDOMNode(this);
      },
      render: function render() {
        var props = this.props;
        var value = props.value,
            prefixCls = props.prefixCls,
            showOk = props.showOk,
            timePicker = props.timePicker,
            renderFooter = props.renderFooter;
    
        var footerEl = null;
        var extraFooter = renderFooter();
        if (props.showToday || timePicker || extraFooter) {
          var _cx;
    
          var nowEl = void 0;
          if (props.showToday) {
            nowEl = _react2["default"].createElement(_TodayButton2["default"], (0, _extends3["default"])({}, props, { value: value }));
          }
          var okBtn = void 0;
          if (showOk === true || showOk !== false && !!props.timePicker) {
            okBtn = _react2["default"].createElement(_OkButton2["default"], props);
          }
          var timePickerBtn = void 0;
          if (!!props.timePicker) {
            timePickerBtn = _react2["default"].createElement(_TimePickerButton2["default"], props);
          }
    
          var footerBtn = void 0;
          if (nowEl || okBtn) {
            footerBtn = _react2["default"].createElement(
              'span',
              { className: prefixCls + '-footer-btn' },
              (0, _mapSelf2["default"])([nowEl, timePickerBtn, okBtn])
            );
          }
    
          var cls = (0, _classnames2["default"])((
            _cx = {}, 
            (0, _defineProperty3["default"])(_cx, prefixCls + '-footer', true), 
            (0, _defineProperty3["default"])(_cx, prefixCls + '-footer-show-ok', okBtn), 
            _cx
          ));
    
          footerEl = _react2["default"].createElement(
            'div',
            { className: cls },
            extraFooter,
            footerBtn
          );
        }
        return footerEl;
      }
    });
    
    // prefixCls，样式类前缀
    // showToday，是否绘制今天按钮
    // showOk，是否绘制确认按钮，且时分秒选择面板存在时绘制
    // timePicker，自定义时分秒选择面板存在时，绘制日期、时分秒面板切换按钮
    // value，影响今天按钮的时间
    // renderFooter，()=>{}绘制额外的尾部元素
    // defaultValue，影响自定义renderFooter节点
    // selectedValue，影响自定义renderFooter节点
    // showDateInput，影响自定义renderFooter节点
    // disabledTime，影响自定义renderFooter节点
    // onSelect，(moment)=>{}，影响自定义renderFooter节点
    // onOk，确认按钮点击执行
    // onToday，今天按钮点击执行
    // onOpenTimePicker，显示时分秒选择面板
    // onCloseTimePicker，隐藏时分秒选择面板
    // local，语言包
    exports["default"] = CalendarFooter;
    module.exports = exports['default'];