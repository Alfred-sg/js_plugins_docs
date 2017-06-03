# FullCalendar、CalendarHeader

FullCalendar绘制日期或月份切换选择面板；CalendarHeader组件可用于FullCalendar组件的头部。

## FullCalendar

### 概述

FullCalendar绘制日期或月份切换选择面板。

### state属性

* type="date|month"，显示日期选择面板还是月份选择面板。

### props属性

* locale，语言包，定义文案
* prefixCls，样式类前缀
* fullscreen，是否全屏显示
* type，类型，"month"或"date"，绘制月份选择面板或日期选择面板
* disabledDate=(moment)=>{}，不可用的日期
* onTypeChange，类型变更时执行函数

* 影响CalendarHeader组件的props属性：
* showHeader，是否显示头部
* showTypeSwitch，是否展示"month"或"date"类型切换控件；若展示，"date"类型下显示月选择下拉框，"month"类型下显示年选择下拉框
* Select，下拉框组件，用于展示年份、月份选择子组件
* headerRender=(value,type,locale)=>{}，自定义头部组件，优先级最高
* headerComponent，自定义头部组件，输入的props属性预定义，优先级次于props.headerRender，高于CalendarHeader
* headerComponents，当以CalendarHeader绘制头部组件时，设定年月下拉框后跟随的自定义组件

* 影响DateTable、MonthTable组件的props属性：
* dateCellRender=(moment,props.value)=>{}，绘制日期节点，优先级最高
* dateCellContentRender=(moment,props.value)=>{}，绘制日期节点的内容，优先级其次；最末是绘制日期节点的日份
* monthCellRender=(moment,locale)=>{}，渲染月份节点，优先级最高
* monthCellContentRender=(moment,locale)=>{}，渲染月份节点内容，优先级其次；最末以月份文案渲染月份节点

### 源码

    'use strict';
    
    Object.defineProperty(exports, "__esModule", {
      value: true
    });
    
    var _extends2 = require('babel-runtime/helpers/extends');
    var _extends3 = _interopRequireDefault(_extends2);
    
    var _react = require('react');
    var _react2 = _interopRequireDefault(_react);
    
    var _DateTable = require('./date/DateTable');
    var _DateTable2 = _interopRequireDefault(_DateTable);
    
    var _MonthTable = require('./month/MonthTable');
    var _MonthTable2 = _interopRequireDefault(_MonthTable);
    
    var _CalendarMixin = require('./mixin/CalendarMixin');
    var _CalendarMixin2 = _interopRequireDefault(_CalendarMixin);
    
    var _CommonMixin = require('./mixin/CommonMixin');
    var _CommonMixin2 = _interopRequireDefault(_CommonMixin);
    
    var _CalendarHeader = require('./full-calendar/CalendarHeader');
    var _CalendarHeader2 = _interopRequireDefault(_CalendarHeader);
    
    function _interopRequireDefault(obj) { return obj && obj.__esModule ? obj : { "default": obj }; }
    
    var FullCalendar = _react2["default"].createClass({
      displayName: 'FullCalendar',
    
      propTypes: {
        defaultType: _react.PropTypes.string,
        type: _react.PropTypes.string,
        prefixCls: _react.PropTypes.string,
        locale: _react.PropTypes.object,
        onTypeChange: _react.PropTypes.func,
        fullscreen: _react.PropTypes.bool,
        monthCellRender: _react.PropTypes.func,
        dateCellRender: _react.PropTypes.func,
        showTypeSwitch: _react.PropTypes.bool,
        Select: _react.PropTypes.func.isRequired,
        headerComponents: _react.PropTypes.array,
        headerComponent: _react.PropTypes.object, // The whole header component
        headerRender: _react.PropTypes.func,
        showHeader: _react.PropTypes.bool,
        disabledDate: _react.PropTypes.func
      },
      mixins: [_CommonMixin2["default"], _CalendarMixin2["default"]],
      getDefaultProps: function getDefaultProps() {
        return {
          defaultType: 'date',
          fullscreen: false,
          showTypeSwitch: true,
          showHeader: true,
          onTypeChange: function onTypeChange() {}
        };
      },
      getInitialState: function getInitialState() {
        var type = void 0;
        if ('type' in this.props) {
          type = this.props.type;
        } else {
          type = this.props.defaultType;
        }
        return {
          type: type
        };
      },
      componentWillReceiveProps: function componentWillReceiveProps(nextProps) {
        if ('type' in nextProps) {
          this.setState({
            type: nextProps.type
          });
        }
      },
      onMonthSelect: function onMonthSelect(value) {
        this.onSelect(value, {
          target: 'month'
        });
      },
      setType: function setType(type) {
        if (!('type' in this.props)) {
          this.setState({
            type: type
          });
        }
        this.props.onTypeChange(type);
      },
      render: function render() {
        var props = this.props;
        var locale = props.locale,
            prefixCls = props.prefixCls,
            fullscreen = props.fullscreen,
            showHeader = props.showHeader,
            headerComponent = props.headerComponent,
            headerRender = props.headerRender,
            disabledDate = props.disabledDate;
        var _state = this.state,
            value = _state.value,
            type = _state.type;
    
        var header = null;
        if (showHeader) {
          if (headerRender) {
            header = headerRender(value, type, locale);
          } else {
            var TheHeader = headerComponent || _CalendarHeader2["default"];
            header = _react2["default"].createElement(TheHeader, (0, _extends3["default"])({
              key: 'calendar-header'
            }, props, {
              prefixCls: prefixCls + '-full',
              type: type,
              value: value,
              onTypeChange: this.setType,
              onValueChange: this.setValue
            }));
          }
        }
    
        var table = type === 'date' ? _react2["default"].createElement(_DateTable2["default"], {
          dateRender: props.dateCellRender,// (moment,props.value)=>{}，绘制日期节点，优先级最高
          contentRender: props.dateCellContentRender,// (moment,props.value)=>{}，绘制日期节点的内容，优先级其次；最末是绘制日期节点的日份
          locale: locale,// 语言包
          prefixCls: prefixCls,// 样式类前缀
          onSelect: this.onSelect,// (moment)=>{}，浮层内日期选择组件点选时执行函数
          value: value,// 获取moment对象的语言包，用于设置星期名
          disabledDate: disabledDate// (moment,props.value)=>{}，判断moment日期节点是否不可点选
        }) : _react2["default"].createElement(_MonthTable2["default"], {
          cellRender: props.monthCellRender,// (moment,locale)=>{}，渲染月份节点，优先级最高
          contentRender: props.monthCellContentRender,// (moment,locale)=>{}，渲染月份节点内容，优先级其次；最末以月份文案渲染月份节点
          locale: locale,// 语言包
          onSelect: this.onMonthSelect,// 月份选择面板点选时执行函数
          prefixCls: prefixCls + '-month-panel',// 样式类前缀
          value: value,// 当前选中的日期对象s
          disabledDate: disabledDate// (moment,props.value)=>{}，判断moment日期节点是否不可点选
        });
    
        var children = [header, _react2["default"].createElement(
          'div',
          { key: 'calendar-body', className: prefixCls + '-calendar-body' },
          table
        )];
    
        var className = [prefixCls + '-full'];
    
        if (fullscreen) {
          className.push(prefixCls + '-fullscreen');
        }
    
        return this.renderRoot({
          children: children,
          className: className.join(' ')
        });
      }
    });
    
    // locale，语言包，定义文案
    // prefixCls，样式类前缀
    // fullscreen，是否全屏显示
    // type，类型，"month"或"date"，绘制月份选择面板或日期选择面板
    // disabledDate=(moment)=>{}，不可用的日期
    // onTypeChange，类型变更时执行函数
    // 
    // 影响CalendarHeader组件的props属性：
    // showHeader，是否显示头部
    // showTypeSwitch，是否展示"month"或"date"类型切换控件；若展示，"date"类型下显示月选择下拉框，"month"类型下显示年选择下拉框
    // Select，下拉框组件，用于展示年份、月份选择子组件
    // headerRender=(value,type,locale)=>{}，自定义头部组件，优先级最高
    // headerComponent，自定义头部组件，输入的props属性预定义，优先级次于props.headerRender，高于CalendarHeader
    // headerComponents，当以CalendarHeader绘制头部组件时，设定年月下拉框后跟随的自定义组件
    // 
    // 影响DateTable、MonthTable组件的props属性：
    // dateCellRender=(moment,props.value)=>{}，绘制日期节点，优先级最高
    // dateCellContentRender=(moment,props.value)=>{}，绘制日期节点的内容，优先级其次；最末是绘制日期节点的日份
    // monthCellRender=(moment,locale)=>{}，渲染月份节点，优先级最高
    // monthCellContentRender=(moment,locale)=>{}，渲染月份节点内容，优先级其次；最末以月份文案渲染月份节点
    exports["default"] = FullCalendar;
    module.exports = exports['default'];




## CalendarHeader

### 概述

CalendarHeader组件可用于绘制FullCalendar组件的头部。

### props属性

* value，选中日期
* locale，语言包
* prefixCls，样式类前缀
* type，类型，"month"或"date"
* showTypeSwitch，是否展示"month"或"date"类型切换控件；若展示，"date"类型下显示月选择下拉框，"month"类型下显示年选择下拉框
* yearSelectOffset，用于设定起始年份，默认为props.value所在年份的前后十年
* yearSelectTotal，用于设定结束年份
* Select，下拉框组件，用于展示年份、月份选择子组件
* onValueChange，年、月下拉框值改变时用于变更日期对象
* onTypeChange，改变父组件的type，即当前组件的props.type
* headerComponents，年月下拉框后跟随的自定义组件

### 源码

    'use strict';
    
    Object.defineProperty(exports, "__esModule", {
      value: true
    });
    
    var _classCallCheck2 = require('babel-runtime/helpers/classCallCheck');
    var _classCallCheck3 = _interopRequireDefault(_classCallCheck2);
    
    var _possibleConstructorReturn2 = require('babel-runtime/helpers/possibleConstructorReturn');
    var _possibleConstructorReturn3 = _interopRequireDefault(_possibleConstructorReturn2);
    
    var _inherits2 = require('babel-runtime/helpers/inherits');
    var _inherits3 = _interopRequireDefault(_inherits2);
    
    var _react = require('react');
    var _react2 = _interopRequireDefault(_react);
    
    function _interopRequireDefault(obj) { return obj && obj.__esModule ? obj : { "default": obj }; }
    
    function noop() {}
    
    var CalendarHeader = function (_Component) {
      (0, _inherits3["default"])(CalendarHeader, _Component);
    
      function CalendarHeader() {
        (0, _classCallCheck3["default"])(this, CalendarHeader);
        return (0, _possibleConstructorReturn3["default"])(this, _Component.apply(this, arguments));
      }
    
      CalendarHeader.prototype.onYearChange = function onYearChange(year) {
        var newValue = this.props.value.clone();
        newValue.year(parseInt(year, 10));
        this.props.onValueChange(newValue);
      };
    
      CalendarHeader.prototype.onMonthChange = function onMonthChange(month) {
        var newValue = this.props.value.clone();
        newValue.month(parseInt(month, 10));
        this.props.onValueChange(newValue);
      };
    
      // 以下拉框形式展示年份选择
      CalendarHeader.prototype.yearSelectElement = function yearSelectElement(year) {
        var _props = this.props,
            yearSelectOffset = _props.yearSelectOffset,
            yearSelectTotal = _props.yearSelectTotal,
            prefixCls = _props.prefixCls,
            Select = _props.Select;
    
        var start = year - yearSelectOffset;
        var end = start + yearSelectTotal;
    
        var options = [];
        for (var index = start; index < end; index++) {
          options.push(_react2["default"].createElement(
            Select.Option,
            { key: '' + index },
            index
          ));
        }
        return _react2["default"].createElement(
          Select,
          {
            className: prefixCls + '-header-year-select',
            onChange: this.onYearChange.bind(this),
            dropdownStyle: { zIndex: 2000 },
            dropdownMenuStyle: { maxHeight: 250, overflow: 'auto', fontSize: 12 },
            optionLabelProp: 'children',
            value: String(year),
            showSearch: false
          },
          options
        );
      };
    
      // 以下拉框形式展示月份选择
      CalendarHeader.prototype.monthSelectElement = function monthSelectElement(month) {
        var props = this.props;
        var localeData = props.value.localeData();
        var t = props.value.clone();
        var prefixCls = props.prefixCls;
    
        var options = [];
        var Select = props.Select;
    
        for (var index = 0; index < 12; index++) {
          t.month(index);
          options.push(_react2["default"].createElement(
            Select.Option,
            { key: '' + index },
            localeData.monthsShort(t)
          ));
        }
    
        return _react2["default"].createElement(
          Select,
          {
            className: prefixCls + '-header-month-select',
            dropdownStyle: { zIndex: 2000 },
            dropdownMenuStyle: { maxHeight: 250, overflow: 'auto', overflowX: 'hidden', fontSize: 12 },
            optionLabelProp: 'children',
            value: String(month),
            showSearch: false,
            onChange: this.onMonthChange.bind(this)
          },
          options
        );
      };
    
      CalendarHeader.prototype.changeTypeToDate = function changeTypeToDate() {
        this.props.onTypeChange('date');
      };
    
      CalendarHeader.prototype.changeTypeToMonth = function changeTypeToMonth() {
        this.props.onTypeChange('month');
      };
    
      CalendarHeader.prototype.render = function render() {
        var _props2 = this.props,
            value = _props2.value,
            locale = _props2.locale,
            prefixCls = _props2.prefixCls,
            type = _props2.type,
            showTypeSwitch = _props2.showTypeSwitch,
            headerComponents = _props2.headerComponents;
    
        var year = value.year();
        var month = value.month();
        var yearSelect = this.yearSelectElement(year);
        var monthSelect = type === 'month' ? null : this.monthSelectElement(month);
        var switchCls = prefixCls + '-header-switcher';
        var typeSwitcher = showTypeSwitch ? _react2["default"].createElement(
          'span',
          { className: switchCls },
          type === 'date' ? _react2["default"].createElement(
            'span',
            { className: switchCls + '-focus' },
            locale.month
          ) : _react2["default"].createElement(
            'span',
            {
              onClick: this.changeTypeToDate.bind(this),
              className: switchCls + '-normal'
            },
            locale.month
          ),
          type === 'month' ? _react2["default"].createElement(
            'span',
            { className: switchCls + '-focus' },
            locale.year
          ) : _react2["default"].createElement(
            'span',
            {
              onClick: this.changeTypeToMonth.bind(this),
              className: switchCls + '-normal'
            },
            locale.year
          )
        ) : null;
    
        return _react2["default"].createElement(
          'div',
          { className: prefixCls + '-header' },
          typeSwitcher,
          monthSelect,
          yearSelect,
          headerComponents
        );
      };
    
      return CalendarHeader;
    }(_react.Component);
    
    CalendarHeader.propTypes = {
      value: _react.PropTypes.object,
      locale: _react.PropTypes.object,
      yearSelectOffset: _react.PropTypes.number,
      yearSelectTotal: _react.PropTypes.number,
      onValueChange: _react.PropTypes.func,
      onTypeChange: _react.PropTypes.func,
      Select: _react.PropTypes.func,
      prefixCls: _react.PropTypes.string,
      type: _react.PropTypes.string,
      showTypeSwitch: _react.PropTypes.bool,
      headerComponents: _react.PropTypes.array
    };
    CalendarHeader.defaultProps = {
      yearSelectOffset: 10,
      yearSelectTotal: 20,
      onValueChange: noop,
      onTypeChange: noop
    };
    
    // value，选中日期
    // locale，语言包
    // prefixCls，样式类前缀
    // type，类型，"month"或"date"
    // showTypeSwitch，是否展示"month"或"date"类型切换控件；若展示，"date"类型下显示月选择下拉框，"month"类型下显示年选择下拉框
    // yearSelectOffset，用于设定起始年份，默认为props.value所在年份的前后十年
    // yearSelectTotal，用于设定结束年份
    // Select，下拉框组件，用于展示年份、月份选择子组件
    // onValueChange，年、月下拉框值改变时用于变更日期对象
    // onTypeChange，改变父组件的type，即当前组件的props.type
    // headerComponents，年月下拉框后跟随的自定义组件
    exports["default"] = CalendarHeader;
    module.exports = exports['default'];