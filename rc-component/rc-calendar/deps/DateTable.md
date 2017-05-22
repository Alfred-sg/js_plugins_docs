# DateTable、DateTHead、DateTBody

DateTable组件调用DateTHead、DateTBody组件绘制浮层日期选择节点。

## DateTable

### 概述

绘制浮层日期选择组件。

### props属性设置

* 影响DateTHead组件：
* prefixCls，样式类前缀
* value，获取moment对象的语言包，用于设置星期名
* showWeekNumber，绘制星期数时，添加th留空

* 影响DateTBody组件：
* prefixCls，样式类前缀
* showWeekNumber，是否绘制日期选择组件中的星期数
* dateRender(moment,props.value)，绘制日期节点，优先级最高
* contentRender(moment,props.value)，绘制日期节点的内容，优先级其次；最末是绘制日期节点的日份
* value，当前选中日期；日期范围组件中，为前置选中日期
* disabledDate(moment,props.value)，判断moment日期节点是否不可点选
* hoverValue，选中的日期节点，优先级高于selectedValue
* selectedValue，选中的日期节点，优先级次于hoverValue
* onSelect(moment)，浮层内日期选择组件点选时执行函数
* onDayHover(moment)，浮层内日期选择组件鼠标移入时执行函数

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
    
    var _DateTHead = require('./DateTHead');
    var _DateTHead2 = _interopRequireDefault(_DateTHead);
    
    var _DateTBody = require('./DateTBody');
    var _DateTBody2 = _interopRequireDefault(_DateTBody);
    
    function _interopRequireDefault(obj) { return obj && obj.__esModule ? obj : { "default": obj }; }
    
    var DateTable = function (_React$Component) {
      (0, _inherits3["default"])(DateTable, _React$Component);
    
      function DateTable() {
        (0, _classCallCheck3["default"])(this, DateTable);
        return (0, _possibleConstructorReturn3["default"])(this, _React$Component.apply(this, arguments));
      }
    
      DateTable.prototype.render = function render() {
        var props = this.props;
        var prefixCls = props.prefixCls;
        return _react2["default"].createElement(
          'table',
          { className: prefixCls + '-table', cellSpacing: '0', role: 'grid' },
          _react2["default"].createElement(_DateTHead2["default"], props),// 星期显示组件
          _react2["default"].createElement(_DateTBody2["default"], props)// 日期点选组件
        );
      };
    
      return DateTable;
    }(_react2["default"].Component);
    
    // 影响DateTHead组件：
    // prefixCls，样式类前缀
    // value，获取moment对象的语言包，用于设置星期
    // showWeekNumber，绘制星期数时，添加th留空
    // 
    // 影响DateTBody组件：
    // prefixCls，样式类前缀
    // showWeekNumber，是否绘制日期选择组件中的星期数
    // dateRender(moment,props.value)，绘制日期节点，优先级最高
    // contentRender(moment,props.value)，绘制日期节点的内容，优先级其次；最末是绘制日期节点的日份
    // value，当前选中日期；日期范围组件中，为前置选中日期
    // disabledDate(moment,props.value)，判断moment日期节点是否不可点选
    // hoverValue，选中的日期节点，优先级高于selectedValue
    // selectedValue，选中的日期节点，优先级次于hoverValue
    // onSelect(moment)，浮层内日期选择组件点选时执行函数
    // onDayHover(moment)，浮层内日期选择组件鼠标移入时执行函数
    exports["default"] = DateTable;
    module.exports = exports['default'];



## DateTHead

### 概述

绘制日期选择组件下，浮层内星期显示组件。

### props属性设置

* prefixCls，样式类前缀
* value，获取moment对象的语言包，用于设置星期
* showWeekNumber，绘制星期数时，添加th留空

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
    
    var _DateConstants = require('./DateConstants');
    var _DateConstants2 = _interopRequireDefault(_DateConstants);
    
    var _moment = require('moment');
    var _moment2 = _interopRequireDefault(_moment);
    
    function _interopRequireDefault(obj) { return obj && obj.__esModule ? obj : { "default": obj }; }
    
    var DateTHead = function (_React$Component) {
      (0, _inherits3["default"])(DateTHead, _React$Component);
    
      function DateTHead() {
        (0, _classCallCheck3["default"])(this, DateTHead);
        return (0, _possibleConstructorReturn3["default"])(this, _React$Component.apply(this, arguments));
      }
    
      DateTHead.prototype.render = function render() {
        var props = this.props;
        var value = props.value;
        var localeData = value.localeData();
        var prefixCls = props.prefixCls;
        var veryShortWeekdays = [];
        var weekDays = [];
        var firstDayOfWeek = localeData.firstDayOfWeek();// 当前语言包的对应起始星期名
        var showWeekNumberEl = void 0;
        var now = (0, _moment2["default"])();
        for (var dateColIndex = 0; dateColIndex < _DateConstants2["default"].DATE_COL_COUNT; dateColIndex++) {
          var index = (firstDayOfWeek + dateColIndex) % _DateConstants2["default"].DATE_COL_COUNT;
          now.day(index);
          // 星期名的极简缩写["Su", "Mo", "Tu", "We", "Th", "Fr", "Sa"]
          veryShortWeekdays[dateColIndex] = localeData.weekdaysMin(now);
          // 星期名的缩写["Sun", "Mon", "Tue", "Wed", "Thu", "Fri", "Sat"]
          weekDays[dateColIndex] = localeData.weekdaysShort(now);
        }
    
        if (props.showWeekNumber) {
          showWeekNumberEl = _react2["default"].createElement(
            'th',
            {
              role: 'columnheader',
              className: prefixCls + '-column-header ' + prefixCls + '-week-number-header'
            },
            _react2["default"].createElement(
              'span',
              { className: prefixCls + '-column-header-inner' },
              'x'
            )
          );
        }
    
        // 星期名节点
        var weekDaysEls = weekDays.map(function (day, xindex) {
          return _react2["default"].createElement(
            'th',
            {
              key: xindex,
              role: 'columnheader',
              title: day,
              className: prefixCls + '-column-header'
            },
            _react2["default"].createElement(
              'span',
              { className: prefixCls + '-column-header-inner' },
              veryShortWeekdays[xindex]
            )
          );
        });
        return _react2["default"].createElement(
          'thead',
          null,
          _react2["default"].createElement(
            'tr',
            { role: 'row' },
            showWeekNumberEl,
            weekDaysEls
          )
        );
      };
    
      return DateTHead;
    }(_react2["default"].Component);
    
    // prefixCls，样式类前缀
    // value，获取moment对象的语言包，用于设置星期
    // showWeekNumber，绘制星期数时，添加th留空
    exports["default"] = DateTHead;
    module.exports = exports['default'];



## DateTBody

### 概述

绘制日期选择组件下，浮层内日期点选组件。

### props属性设置

* prefixCls，样式类前缀
* showWeekNumber，是否绘制日期选择组件中的星期节点，即["Su", "Mo", "Tu", "We", "Th", "Fr", "Sa"]
* dateRender(moment,props.value)，绘制日期节点，优先级最高
* contentRender(moment,props.value)，绘制日期节点的内容，优先级其次；最末是绘制日期节点的日份
* value，当前选中日期；日期范围组件中，为前置选中日期
* disabledDate(moment,props.value)，判断moment日期节点是否不可点选
* hoverValue，选中的日期节点，优先级高于selectedValue
* selectedValue，选中的日期节点，优先级次于hoverValue
* onSelect(moment)，浮层内日期选择组件点选时执行函数
* onDayHover(moment)，浮层内日期选择组件鼠标移入时执行函数

### 源码

    'use strict';
    
    Object.defineProperty(exports, "__esModule", {
      value: true
    });
    
    var _react = require('react');
    var _react2 = _interopRequireDefault(_react);
    
    var _DateConstants = require('./DateConstants');
    var _DateConstants2 = _interopRequireDefault(_DateConstants);
    
    var _util = require('../util/');
    
    function _interopRequireDefault(obj) { return obj && obj.__esModule ? obj : { "default": obj }; }
    
    function isSameDay(one, two) {
      return one && two && one.isSame(two, 'day');
    }
    
    // 时间对象current的年月份小于today
    function beforeCurrentMonthYear(current, today) {
      if (current.year() < today.year()) {
        return 1;
      }
      return current.year() === today.year() && current.month() < today.month();
    }
    
    // 时间对象current的年月份大于today
    function afterCurrentMonthYear(current, today) {
      if (current.year() > today.year()) {
        return 1;
      }
      return current.year() === today.year() && current.month() > today.month();
    }
    
    // 获取日期节点的key键
    function getIdFromDate(date) {
      return 'rc-calendar-' + date.year() + '-' + date.month() + '-' + date.date();
    }
    
    var DateTBody = _react2["default"].createClass({
      displayName: 'DateTBody',
    
      propTypes: {
        contentRender: _react.PropTypes.func,
        dateRender: _react.PropTypes.func,
        disabledDate: _react.PropTypes.func,
        prefixCls: _react.PropTypes.string,
        selectedValue: _react.PropTypes.oneOfType([_react.PropTypes.object, _react.PropTypes.arrayOf(_react.PropTypes.object)]),
        value: _react.PropTypes.object,
        hoverValue: _react.PropTypes.any,
        showWeekNumber: _react.PropTypes.bool
      },
    
      getDefaultProps: function getDefaultProps() {
        return {
          hoverValue: []
        };
      },
      render: function render() {
        var props = this.props;
        var contentRender = props.contentRender,
            prefixCls = props.prefixCls,
            selectedValue = props.selectedValue,
            value = props.value,
            showWeekNumber = props.showWeekNumber,
            dateRender = props.dateRender,
            disabledDate = props.disabledDate,
            hoverValue = props.hoverValue;
    
        var iIndex = void 0;
        var jIndex = void 0;
        var current = void 0;
        var dateTable = [];
        var today = (0, _util.getTodayTime)(value);
        var cellClass = prefixCls + '-cell';
        var weekNumberCellClass = prefixCls + '-week-number-cell';
        var dateClass = prefixCls + '-date';
        var todayClass = prefixCls + '-today';
        var selectedClass = prefixCls + '-selected-day';
        var selectedDateClass = prefixCls + '-selected-date'; // do not move with mouse operation
        var inRangeClass = prefixCls + '-in-range-cell';
        var lastMonthDayClass = prefixCls + '-last-month-cell';
        var nextMonthDayClass = prefixCls + '-next-month-btn-day';
        var disabledClass = prefixCls + '-disabled-cell';
        var firstDisableClass = prefixCls + '-disabled-cell-first-of-row';
        var lastDisableClass = prefixCls + '-disabled-cell-last-of-row';
    
        var month1 = value.clone();
        month1.date(1);// 将moment对象设置为所在月份的首日
        var day = month1.day();// props.value在月份中的日数
        // 获取日期选择面板该显示的上个月日数
        var lastMonthDiffDay = (day + 7 - value.localeData().firstDayOfWeek()) % 7;
        var lastMonth1 = month1.clone();
        lastMonth1.add(0 - lastMonthDiffDay, 'days');
    
        // 将42个日期塞入dateTable日期选择组件中
        var passed = 0;
        for (iIndex = 0; iIndex < _DateConstants2["default"].DATE_ROW_COUNT; iIndex++) {
          for (jIndex = 0; jIndex < _DateConstants2["default"].DATE_COL_COUNT; jIndex++) {
            current = lastMonth1;
            if (passed) {
              current = current.clone();
              current.add(passed, 'days');
            }
            dateTable.push(current);
            passed++;
          }
        }
    
        var tableHtml = [];
        passed = 0;
    
        for (iIndex = 0; iIndex < _DateConstants2["default"].DATE_ROW_COUNT; iIndex++) {
          var weekNumberCell = void 0;
          var dateCells = [];
          if (showWeekNumber) {
            // 绘制星期数
            weekNumberCell = _react2["default"].createElement(
              'td',
              {
                key: dateTable[passed].week(),
                role: 'gridcell',
                className: weekNumberCellClass
              },
              dateTable[passed].week()
            );
          }
    
          for (jIndex = 0; jIndex < _DateConstants2["default"].DATE_COL_COUNT; jIndex++) {
            var next = null;
            var last = null;
            current = dateTable[passed];
            // last、next日期用于判断不可点选的日期是否起始、结束不可点选节点
            if (jIndex < _DateConstants2["default"].DATE_COL_COUNT - 1) {
              next = dateTable[passed + 1];
            }
            if (jIndex > 0) {
              last = dateTable[passed - 1];
            }
            var cls = cellClass;
            var disabled = false;
            var selected = false;
    
            if (isSameDay(current, today)) {
              cls += ' ' + todayClass;
            }
    
            // 日期选择面板正在绘制的日期节点current的年月份小于已选中日期节点value，绘制样式用，非激活月份置灰
            var isBeforeCurrentMonthYear = beforeCurrentMonthYear(current, value);
            var isAfterCurrentMonthYear = afterCurrentMonthYear(current, value);
    
            // 日期范围选择器下，赋值selected标识符、样式cls以激活选中节点、选中节点间的样式
            if (selectedValue && Array.isArray(selectedValue)) {
              // 日期选中范围，props.hoverValue优先级高于props.selectedValue
              var rangeValue = hoverValue.length ? hoverValue : selectedValue;
              if (!isBeforeCurrentMonthYear && !isAfterCurrentMonthYear) {
                var startValue = rangeValue[0];
                var endValue = rangeValue[1];
                if (startValue) {
                  // 是否选中节点
                  if (isSameDay(current, startValue)) {
                    selected = true;
                  }
                }
                if (startValue && endValue) {
                  // 是否选中节点
                  if (isSameDay(current, endValue)) {
                    selected = true;
                  // 当前绘制的日期节点在选中的前置、后置日期之间，添加激活样式
                  } else if (current.isAfter(startValue, 'day') && current.isBefore(endValue, 'day')) {
                    cls += ' ' + inRangeClass;
                  }
                }
              }
            // 日期单选选择器下，赋值selected标识符、样式cls以激活选中节点的样式
            } else if (isSameDay(current, value)) {
              selected = true;
            }
    
            // 日期单选选择器下，赋值样式cls以激活选中节点的样式
            if (isSameDay(current, selectedValue)) {
              cls += ' ' + selectedDateClass;
            }
    
            // 绘制日期节点时，赋值样式cls已激活置灰月份日期节点
            if (isBeforeCurrentMonthYear) {
              cls += ' ' + lastMonthDayClass;
            }
            if (isAfterCurrentMonthYear) {
              cls += ' ' + nextMonthDayClass;
            }
    
            // 绘制日期节点时，赋值样式cls激活首个、尾个不可点选的日期节点
            if (disabledDate) {
              if (disabledDate(current, value)) {
                disabled = true;
    
                if (!last || !disabledDate(last, value)) {
                  cls += ' ' + firstDisableClass;
                }
    
                if (!next || !disabledDate(next, value)) {
                  cls += ' ' + lastDisableClass;
                }
              }
            }
    
            // 绘制日期节点时，赋值样式cls激活选中的日期节点
            if (selected) {
              cls += ' ' + selectedClass;
            }
    
            // 绘制日期节点时，赋值样式cls激活不可点选的日期节点
            if (disabled) {
              cls += ' ' + disabledClass;
            }
    
            var dateHtml = void 0;
            if (dateRender) {
              dateHtml = dateRender(current, value);
            } else {
              var content = contentRender ? contentRender(current, value) : current.date();
              dateHtml = _react2["default"].createElement(
                'div',
                {
                  key: getIdFromDate(current),
                  className: dateClass,
                  'aria-selected': selected,
                  'aria-disabled': disabled
                },
                content
              );
            }
    
            dateCells.push(_react2["default"].createElement(
              'td',
              {
                key: passed,
                onClick: disabled ? undefined : props.onSelect.bind(null, current),
                onMouseEnter: disabled ? undefined : props.onDayHover && props.onDayHover.bind(null, current) || undefined,
                role: 'gridcell',
                title: (0, _util.getTitleString)(current), 
                className: cls
              },
              dateHtml
            ));
    
            passed++;
          }
          tableHtml.push(_react2["default"].createElement(
            'tr',
            {
              key: iIndex,
              role: 'row'
            },
            weekNumberCell,
            dateCells
          ));
        }
        return _react2["default"].createElement(
          'tbody',
          { className: prefixCls + '-tbody' },
          tableHtml
        );
      }
    });
    
    // prefixCls，样式类前缀
    // showWeekNumber，是否绘制星期数
    // dateRender(moment,props.value)，绘制日期节点，优先级最高
    // contentRender(moment,props.value)，绘制日期节点的内容，优先级其次；最末是绘制日期节点的日份
    // value，当前选中日期；日期范围组件中，为前置选中日期
    // disabledDate(moment,props.value)，判断moment日期节点是否不可点选
    // hoverValue，选中的日期节点，优先级高于selectedValue
    // selectedValue，选中的日期节点，优先级次于hoverValue
    // onSelect(moment)，浮层内日期选择组件点选时执行函数
    // onDayHover(moment)，浮层内日期选择组件鼠标移入时执行函数
    exports["default"] = DateTBody;
    module.exports = exports['default'];


## DateConstants

常数

    "use strict";
    
    Object.defineProperty(exports, "__esModule", {
      value: true
    });
    exports["default"] = {
      DATE_ROW_COUNT: 6,
      DATE_COL_COUNT: 7
    };
    module.exports = exports['default'];