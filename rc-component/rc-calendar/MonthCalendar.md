# MonthCalendar

## 概述

绘制年月选择面板。

## 源码

    'use strict';
    
    Object.defineProperty(exports, "__esModule", {
      value: true
    });
    
    var _react = require('react');
    var _react2 = _interopRequireDefault(_react);
    
    var _MonthPanel = require('./month/MonthPanel');
    var _MonthPanel2 = _interopRequireDefault(_MonthPanel);
    
    var _CalendarMixin = require('./mixin/CalendarMixin');
    var _CalendarMixin2 = _interopRequireDefault(_CalendarMixin);
    
    var _CommonMixin = require('./mixin/CommonMixin');
    var _CommonMixin2 = _interopRequireDefault(_CommonMixin);
    
    var _KeyCode = require('rc-util/lib/KeyCode');
    var _KeyCode2 = _interopRequireDefault(_KeyCode);
    
    function _interopRequireDefault(obj) { return obj && obj.__esModule ? obj : { "default": obj }; }
    
    var MonthCalendar = _react2["default"].createClass({
      displayName: 'MonthCalendar',
    
      propTypes: {
        monthCellRender: _react.PropTypes.func,
        dateCellRender: _react.PropTypes.func
      },
      mixins: [_CommonMixin2["default"], _CalendarMixin2["default"]],
    
      onKeyDown: function onKeyDown(event) {
        var keyCode = event.keyCode;
        var ctrlKey = event.ctrlKey || event.metaKey;
        var stateValue = this.state.value;
        var value = stateValue;
        switch (keyCode) {
          case _KeyCode2["default"].DOWN:
            value = stateValue.clone();
            value.add(3, 'months');
            break;
          case _KeyCode2["default"].UP:
            value = stateValue.clone();
            value.add(-3, 'months');
            break;
          case _KeyCode2["default"].LEFT:
            value = stateValue.clone();
            if (ctrlKey) {
              value.add(-1, 'years');
            } else {
              value.add(-1, 'months');
            }
            break;
          case _KeyCode2["default"].RIGHT:
            value = stateValue.clone();
            if (ctrlKey) {
              value.add(1, 'years');
            } else {
              value.add(1, 'months');
            }
            break;
          case _KeyCode2["default"].ENTER:
            this.onSelect(stateValue);
            event.preventDefault();
            return 1;
          default:
            return undefined;
        }
        if (value !== stateValue) {
          this.setValue(value);
          event.preventDefault();
          return 1;
        }
      },
      render: function render() {
        var props = this.props;
        var children = _react2["default"].createElement(_MonthPanel2["default"], {
          locale: props.locale,// 语言包，决定文案
          disabledDate: props.disabledDate,// (moment)=>{}，判断不可选的日期
          style: { position: 'relative' },
          value: this.state.value,// 选中的日期对象
          cellRender: props.monthCellRender,// (moment,locale)=>{}，渲染月份节点，优先级最高
          contentRender: props.monthCellContentRender,// (moment,locale)=>{}，渲染月份节点内容，优先级其次；最末以月份文案渲染月份节点
          rootPrefixCls: props.prefixCls,// 样式类前缀
          onChange: this.setValue,// 年份选择面板点选时执行函数
          onSelect: this.onSelect// 月份选择面板点选时执行函数
        });
        return this.renderRoot({
          children: children
        });
      }
    });
    
    exports["default"] = MonthCalendar;
    module.exports = exports['default'];