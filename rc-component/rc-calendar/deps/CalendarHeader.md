# CalendarHeader、YearPanel、DecadePanel、MonthPanel、MonthTable

CalendarHeader组件调用YearPanel、MonthPanel组件渲染日期选择面板的年、月切换头；通过切换头内按键，渲染年份、月份选择面板，即YearPanel、MonthPanel组件。

## CalendarHeader

### 概述

渲染日期选择面板的年、月切换头；通过切换头内按键，渲染年份、月份选择面板。

### state属性

* showMonthPanel，显示月份选择面板；1-显示，0-隐藏
* showYearPanel，显示年份选择面板；1-显示，0-隐藏

### props属性

* enableNext，下一年、下一月点选按钮是否可用
* enablePrev，上一年、上一月点选按钮是否可用
* prefixCls，样式类前缀
* locale，语言包，上一年、上一月、下一月、下一年点选按钮文案
* value，当前选中的日期
* showTimePicker，时分秒选择面板显示中状态标识
 
* 影响YearPanel年份选择面板、MonthPanel月份选择面板的props属性：
* locale，语言包，决定文案
* value，当前选中的日期
* prefixCls，样式类前缀

### 源码

    'use strict';
    
    Object.defineProperty(exports, "__esModule", {
      value: true
    });
    
    var _react = require('react');
    var _react2 = _interopRequireDefault(_react);
    
    var _MonthPanel = require('../month/MonthPanel');
    var _MonthPanel2 = _interopRequireDefault(_MonthPanel);
    
    var _YearPanel = require('../year/YearPanel');
    var _YearPanel2 = _interopRequireDefault(_YearPanel);
    
    var _mapSelf = require('rc-util/lib/Children/mapSelf');
    var _mapSelf2 = _interopRequireDefault(_mapSelf);
    
    function _interopRequireDefault(obj) { return obj && obj.__esModule ? obj : { "default": obj }; }
    
    // 切换至上一月、下一月，并调用props.onValueChange改变选中的日期对象
    function goMonth(direction) {
      var next = this.props.value.clone();
      next.add(direction, 'months');
      this.props.onValueChange(next);
    }
    
    // 切换至上一年、下一年，并调用props.onValueChange改变选中的日期对象
    function goYear(direction) {
      var next = this.props.value.clone();
      next.add(direction, 'years');
      this.props.onValueChange(next);
    }
    
    var CalendarHeader = _react2["default"].createClass({
      displayName: 'CalendarHeader',
    
      propTypes: {
        enablePrev: _react.PropTypes.any,
        enableNext: _react.PropTypes.any,
        prefixCls: _react.PropTypes.string,
        showTimePicker: _react.PropTypes.bool,
        locale: _react.PropTypes.object,
        value: _react.PropTypes.object,
        onValueChange: _react.PropTypes.func
      },
    
      getDefaultProps: function getDefaultProps() {
        return {
          enableNext: 1,
          enablePrev: 1
        };
      },
      getInitialState: function getInitialState() {
        this.nextMonth = goMonth.bind(this, 1);
        this.previousMonth = goMonth.bind(this, -1);
        this.nextYear = goYear.bind(this, 1);
        this.previousYear = goYear.bind(this, -1);
        return {};
      },
    
      // 年份、月份选择面板点选时执行函数，并调用props.onValueChange改变选中的日期对象
      onSelect: function onSelect(value) {
        this.setState({
          showMonthPanel: 0,
          showYearPanel: 0
        });
        this.props.onValueChange(value);
      },
    
      // 渲染选中的年、月份节点
      monthYearElement: function monthYearElement(showTimePicker) {
        var props = this.props;
        var prefixCls = props.prefixCls;
        var locale = props.locale;
        var value = props.value;
        var monthBeforeYear = locale.monthBeforeYear;
        var selectClassName = prefixCls + '-' + (monthBeforeYear ? 'my-select' : 'ym-select');
        var year = _react2["default"].createElement(
          'a',
          {
            className: prefixCls + '-year-select',
            role: 'button',
            onClick: showTimePicker ? null : this.showYearPanel,// 显示年份选择面板
            title: locale.yearSelect
          },
          value.format(locale.yearFormat)
        );
        var month = _react2["default"].createElement(
          'a',
          {
            className: prefixCls + '-month-select',
            role: 'button',
            onClick: showTimePicker ? null : this.showMonthPanel,// 显示月份选择面板
            title: locale.monthSelect
          },
          value.format(locale.monthFormat)
        );
        var day = void 0;
        if (showTimePicker) {
          day = _react2["default"].createElement(
            'a',
            {
              className: prefixCls + '-day-select',
              role: 'button'
            },
            value.format(locale.dayFormat)
          );
        }
        var my = [];
        if (monthBeforeYear) {
          my = [month, day, year];
        } else {
          my = [year, month, day];
        }
        return _react2["default"].createElement(
          'span',
          { className: selectClassName },
          (0, _mapSelf2["default"])(my)
        );
      },
    
      // (condition,el)=>{}，当condition为真值时，绘制el
      showIf: function showIf(condition, el) {
        return condition ? el : null;
      },
    
      // 显示月份选择面板
      showMonthPanel: function showMonthPanel() {
        this.setState({
          showMonthPanel: 1,
          showYearPanel: 0
        });
      },
    
      // 显示年份选择面板
      showYearPanel: function showYearPanel() {
        this.setState({
          showMonthPanel: 0,
          showYearPanel: 1
        });
      },
      render: function render() {
        var props = this.props;
        var enableNext = props.enableNext,
            enablePrev = props.enablePrev,
            prefixCls = props.prefixCls,
            locale = props.locale,
            value = props.value,
            showTimePicker = props.showTimePicker;
    
        var state = this.state;
        var PanelClass = null;
        // 显示月份选择面板
        if (state.showMonthPanel) {
          PanelClass = _MonthPanel2["default"];
    
        // 显示年份选择面板
        } else if (state.showYearPanel) {
          PanelClass = _YearPanel2["default"];
        }
    
        // 年份或月份选择面板
        var panel = void 0;
        if (PanelClass) {
          panel = _react2["default"].createElement(PanelClass, {
            locale: locale,// 语言包，决定文案
            defaultValue: value,// 当前选中的日期，影响选中的年份
            rootPrefixCls: prefixCls,// 样式类前缀
            onSelect: this.onSelect// 年份、月份选择面板点选时执行函数
          });
        }
    
        return _react2["default"].createElement(
          'div',
          { className: prefixCls + '-header' },
          _react2["default"].createElement(
            'div',
            { style: { position: 'relative' } },
    
            // 上一年、上一月点选按钮
            this.showIf(enablePrev && !showTimePicker, _react2["default"].createElement('a', {
              className: prefixCls + '-prev-year-btn',
              role: 'button',
              onClick: this.previousYear,
              title: locale.previousYear
            })),
            this.showIf(enablePrev && !showTimePicker, _react2["default"].createElement('a', {
              className: prefixCls + '-prev-month-btn',
              role: 'button',
              onClick: this.previousMonth,
              title: locale.previousMonth
            })),
    
            // 显示选中的年、月，点击时切换年、月份选择面板
            this.monthYearElement(showTimePicker),
    
            // 下一年、下一月点选按钮
            this.showIf(enableNext && !showTimePicker, _react2["default"].createElement('a', {
              className: prefixCls + '-next-month-btn',
              onClick: this.nextMonth,
              title: locale.nextMonth
            })),
            this.showIf(enableNext && !showTimePicker, _react2["default"].createElement('a', {
              className: prefixCls + '-next-year-btn',
              onClick: this.nextYear,
              title: locale.nextYear
            }))
          ),
    
          // 年份或月份选择面板
          panel
        );
      }
    });
    
    // enableNext，下一年、下一月点选按钮是否显示
    // enablePrev，上一年、上一月点选按钮是否显示
    // prefixCls，样式类前缀
    // locale，语言包，上一年、上一月、下一月、下一年点选按钮文案
    // value，当前选中的日期
    // showTimePicker，时分秒选择面板显示中状态标识
    // onValueChange，改变选中的日期对象，年份、月份选择面板点选时执行
    // 
    // 影响YearPanel年份选择面板、MonthPanel月份选择面板的props属性：
    // locale，语言包，决定文案
    // value，当前选中的日期
    // prefixCls，样式类前缀
    exports["default"] = CalendarHeader;
    module.exports = exports['default'];



## YearPanel

### 概述

渲染年份选择面板。 

### state属性

* value，当前选中的日期对象
* showDecadePanel，显示十年选择面板；1-显示，0-隐藏

### props属性 

* locale，语言包，决定按钮文案
* rootPrefixCls，样式类前缀
* value，选中的年份，优先级高于props.defaultValue
* defaultValue，选中的年份
* onSelect，更改选中的年份
 
* 影响DecadePanel十年选择面板的props属性：
* locale，语言包，决定文案
* rootPrefixCls，样式类前缀

### 源码 

    'use strict';
    
    Object.defineProperty(exports, "__esModule", {
      value: true
    });
    
    var _defineProperty2 = require('babel-runtime/helpers/defineProperty');
    var _defineProperty3 = _interopRequireDefault(_defineProperty2);
    
    var _classCallCheck2 = require('babel-runtime/helpers/classCallCheck');
    var _classCallCheck3 = _interopRequireDefault(_classCallCheck2);
    
    var _possibleConstructorReturn2 = require('babel-runtime/helpers/possibleConstructorReturn');
    var _possibleConstructorReturn3 = _interopRequireDefault(_possibleConstructorReturn2);
    
    var _inherits2 = require('babel-runtime/helpers/inherits');
    var _inherits3 = _interopRequireDefault(_inherits2);
    
    var _react = require('react');
    var _react2 = _interopRequireDefault(_react);
    
    var _classnames = require('classnames');
    var _classnames2 = _interopRequireDefault(_classnames);
    
    var _DecadePanel = require('../decade/DecadePanel');
    var _DecadePanel2 = _interopRequireDefault(_DecadePanel);
    
    function _interopRequireDefault(obj) { return obj && obj.__esModule ? obj : { "default": obj }; }
    
    var ROW = 4;
    var COL = 3;
    
    // 切换年份面板，跳到上一个十年或下一个十年
    function goYear(direction) {
      var value = this.state.value.clone();
      value.add(direction, 'year');
      this.setState({
        value: value
      });
    }
    
    // 年份选择面板点击年份，更改选中的年份
    function chooseYear(year) {
      var value = this.state.value.clone();
      value.year(year);
      value.month(this.state.value.month());
      this.props.onSelect(value);
    }
    
    var YearPanel = function (_React$Component) {
      (0, _inherits3["default"])(YearPanel, _React$Component);
    
      function YearPanel(props) {
        (0, _classCallCheck3["default"])(this, YearPanel);
    
        var _this = (0, _possibleConstructorReturn3["default"])(this, _React$Component.call(this, props));
    
        _this.prefixCls = props.rootPrefixCls + '-year-panel';
        _this.state = {
          value: props.value || props.defaultValue
        };
        _this.nextDecade = goYear.bind(_this, 10);// 切换到前一个年份面板，即上一个十年
        _this.previousDecade = goYear.bind(_this, -10);// 切换到后一个年份面板，即下一个十年
        ['showDecadePanel', 'onDecadePanelSelect'].forEach(function (method) {
          _this[method] = _this[method].bind(_this);
        });
        return _this;
      }
    
      // 隐藏十年切换面板，显示年份选择面板；并更改this.state.value，改变年份选择面板选中值
      YearPanel.prototype.onDecadePanelSelect = function onDecadePanelSelect(current) {
        this.setState({
          value: current,
          showDecadePanel: 0
        });
      };
    
      // 年份节点数据，显示12个年，前后两个置灰
      YearPanel.prototype.years = function years() {
        var value = this.state.value;
        var currentYear = value.year();
        var startYear = parseInt(currentYear / 10, 10) * 10;
        var previousYear = startYear - 1;
        var years = [];
        var index = 0;
        for (var rowIndex = 0; rowIndex < ROW; rowIndex++) {
          years[rowIndex] = [];
          for (var colIndex = 0; colIndex < COL; colIndex++) {
            var year = previousYear + index;
            var content = String(year);
            years[rowIndex][colIndex] = {
              content: content,
              year: year,
              title: content
            };
            index++;
          }
        }
        return years;
      };
    
      // 显示十年切换面板，隐藏年份选择面板
      YearPanel.prototype.showDecadePanel = function showDecadePanel() {
        this.setState({
          showDecadePanel: 1
        });
      };
    
      YearPanel.prototype.render = function render() {
        var _this2 = this;
    
        var props = this.props;
        var value = this.state.value;
        var locale = props.locale;
        var years = this.years();
        var currentYear = value.year();
        var startYear = parseInt(currentYear / 10, 10) * 10;// 可选的起始年份
        var endYear = startYear + 9;// 可选的结束年份
        var prefixCls = this.prefixCls;
    
        // 年份节点，置灰的节点用于切换年份选择面板，上翻一页、下翻一页；可选的年份更改当前年份
        var yeasEls = years.map(function (row, index) {
          var tds = row.map(function (yearData) {
            var _classNameMap;
    
            // 年份节点的样式类
            var classNameMap = (_classNameMap = {}, 
              (0, _defineProperty3["default"])(_classNameMap, prefixCls + '-cell', 1), 
              (0, _defineProperty3["default"])(_classNameMap, prefixCls + '-selected-cell', yearData.year === currentYear), 
              (0, _defineProperty3["default"])(_classNameMap, prefixCls + '-last-decade-cell', yearData.year < startYear), 
              (0, _defineProperty3["default"])(_classNameMap, prefixCls + '-next-decade-cell', yearData.year > endYear), 
              _classNameMap
            );
    
            var clickHandler = void 0;
            if (yearData.year < startYear) {
              clickHandler = _this2.previousDecade;// 切换到前一个年份面板，即上一个十年
            } else if (yearData.year > endYear) {
              clickHandler = _this2.nextDecade;// 切换到后一个年份面板，即下一个十年
            } else {
              clickHandler = chooseYear.bind(_this2, yearData.year);// 选择年份
            }
    
            return _react2["default"].createElement(
              'td',
              {
                role: 'gridcell',
                title: yearData.title,
                key: yearData.content,
                onClick: clickHandler,
                className: (0, _classnames2["default"])(classNameMap)
              },
              _react2["default"].createElement(
                'a',
                {
                  className: prefixCls + '-year'
                },
                yearData.content
              )
            );
          });
    
          return _react2["default"].createElement(
            'tr',
            { key: index, role: 'row' },
            tds
          );
        });
    
        // 十年选择面板，this.state.showDecadePanel为真显示
        var decadePanel = void 0;
        if (this.state.showDecadePanel) {
          decadePanel = _react2["default"].createElement(_DecadePanel2["default"], {
            locale: locale,// 语言包，决定文案
            value: value,// 选中的年份，优先级高于props.defaultValue
            rootPrefixCls: props.rootPrefixCls,// 样式类前缀
            onSelect: this.onDecadePanelSelect// 隐藏十年切换面板，显示年份选择面板；并更改年份选择面板选中值
          });
        }
    
        return _react2["default"].createElement(
          'div',
          { className: this.prefixCls },
          _react2["default"].createElement(
            'div',
            null,
            _react2["default"].createElement(
              'div',
    
              // 上十年切换按钮
              { className: prefixCls + '-header' },
              _react2["default"].createElement('a', {
                className: prefixCls + '-prev-decade-btn',
                role: 'button',
                onClick: this.previousDecade,
                title: locale.previousDecade
              }),
    
              _react2["default"].createElement(
                'a',
                {
                  className: prefixCls + '-decade-select',
                  role: 'button',
                  onClick: this.showDecadePanel,// 切换到十年选择面板
                  title: locale.decadeSelect
                },
                _react2["default"].createElement(
                  'span',
                  { className: prefixCls + '-decade-select-content' },
                  startYear,
                  '-',
                  endYear
                ),
                _react2["default"].createElement(
                  'span',
                  { className: prefixCls + '-decade-select-arrow' },
                  'x'
                )
              ),
    
              // 下十年切换按钮
              _react2["default"].createElement('a', {
                className: prefixCls + '-next-decade-btn',
                role: 'button',
                onClick: this.nextDecade,
                title: locale.nextDecade
              })
            ),
            _react2["default"].createElement(
              'div',
              { className: prefixCls + '-body' },
              _react2["default"].createElement(
                'table',
                { className: prefixCls + '-table', cellSpacing: '0', role: 'grid' },
                _react2["default"].createElement(
                  'tbody',
                  { className: prefixCls + '-tbody' },
                  yeasEls
                )
              )
            )
          ),
    
          // 十年选择面板，由this.state.showDecadePanel决定显隐
          decadePanel
        );
      };
    
      return YearPanel;
    }(_react2["default"].Component);
    
    exports["default"] = YearPanel;
    
    YearPanel.propTypes = {
      rootPrefixCls: _react.PropTypes.string,
      value: _react.PropTypes.object,
      defaultValue: _react.PropTypes.object
    };
    
    YearPanel.defaultProps = {
      onSelect: function onSelect() {}
    };
    
    // locale，语言包，决定按钮文案
    // rootPrefixCls，样式类前缀
    // value，选中的年份，优先级高于props.defaultValue
    // defaultValue，选中的年份
    // onSelect，更改选中的年份
    // 
    // 影响DecadePanel十年选择面板的props属性：
    // locale，语言包，决定文案
    // rootPrefixCls，样式类前缀
    module.exports = exports['default'];



## DecadePanel

### 概述

渲染十年选择面板，用于以十年为单位选择年份。

### state属性 

* value，当前选中的日期对象

### props属性 

* locale，语言包，决定文案
* rootPrefixCls，样式类前缀
* value，选中的年份，优先级高于props.defaultValue
* defaultValue，选中的年份
* onSelect，隐藏十年切换面板，显示年份选择面板；并更改年份选择面板选中值

### 源码 

    'use strict';
    
    Object.defineProperty(exports, "__esModule", {
      value: true
    });
    
    var _defineProperty2 = require('babel-runtime/helpers/defineProperty');
    var _defineProperty3 = _interopRequireDefault(_defineProperty2);
    
    var _classCallCheck2 = require('babel-runtime/helpers/classCallCheck');
    var _classCallCheck3 = _interopRequireDefault(_classCallCheck2);
    
    var _possibleConstructorReturn2 = require('babel-runtime/helpers/possibleConstructorReturn');
    var _possibleConstructorReturn3 = _interopRequireDefault(_possibleConstructorReturn2);
    
    var _inherits2 = require('babel-runtime/helpers/inherits');
    var _inherits3 = _interopRequireDefault(_inherits2);
    
    var _react = require('react');
    var _react2 = _interopRequireDefault(_react);
    
    var _classnames = require('classnames');
    var _classnames2 = _interopRequireDefault(_classnames);
    
    function _interopRequireDefault(obj) { return obj && obj.__esModule ? obj : { "default": obj }; }
    
    var ROW = 4;
    var COL = 3;
    
    // 切换十年选择面板，跳到上一百年或下一百年
    function goYear(direction) {
      var next = this.state.value.clone();
      next.add(direction, 'years');
      this.setState({
        value: next
      });
    }
    
    // 十年选择面板点击十年选择节点，更改选中的年份为选中十年的首年
    function chooseDecade(year, event) {
      var next = this.state.value.clone();
      next.year(year);
      next.month(this.state.value.month());
      this.props.onSelect(next);// 隐藏十年切换面板，显示年份选择面板；并更改年份选择面板选中值
      event.preventDefault();
    }
    
    var DecadePanel = function (_React$Component) {
      (0, _inherits3["default"])(DecadePanel, _React$Component);
    
      function DecadePanel(props) {
        (0, _classCallCheck3["default"])(this, DecadePanel);
    
        var _this = (0, _possibleConstructorReturn3["default"])(this, _React$Component.call(this, props));
    
        _this.state = {
          value: props.value || props.defaultValue
        };
    
        _this.prefixCls = props.rootPrefixCls + '-decade-panel';
        _this.nextCentury = goYear.bind(_this, 100);// 切换到上一个十年面板，即上一百年
        _this.previousCentury = goYear.bind(_this, -100);// 切换到下一个十年面板，即下一百年
        return _this;
      }
    
      DecadePanel.prototype.render = function render() {
        var _this2 = this;
    
        var value = this.state.value;
        var locale = this.props.locale;
        var currentYear = value.year();
        var startYear = parseInt(currentYear / 100, 10) * 100;
        var preYear = startYear - 10;// 可选的起始十年
        var endYear = startYear + 99;// 可选的结束十年
        var decades = [];
        var index = 0;
        var prefixCls = this.prefixCls;
    
        for (var rowIndex = 0; rowIndex < ROW; rowIndex++) {
          decades[rowIndex] = [];
          for (var colIndex = 0; colIndex < COL; colIndex++) {
            var startDecade = preYear + index * 10;
            var endDecade = preYear + index * 10 + 9;
            decades[rowIndex][colIndex] = {
              startDecade: startDecade,
              endDecade: endDecade
            };
            index++;
          }
        }
    
        var decadesEls = decades.map(function (row, decadeIndex) {
          var tds = row.map(function (decadeData) {
            var _classNameMap;
    
            var dStartDecade = decadeData.startDecade;
            var dEndDecade = decadeData.endDecade;
            var isLast = dStartDecade < startYear;
            var isNext = dEndDecade > endYear;
    
            // 十年选择节点的样式类
            var classNameMap = (
              _classNameMap = {}, 
              (0, _defineProperty3["default"])(_classNameMap, prefixCls + '-cell', 1), 
              (0, _defineProperty3["default"])(_classNameMap, prefixCls + '-selected-cell', dStartDecade <= currentYear && currentYear <= dEndDecade), 
              (0, _defineProperty3["default"])(_classNameMap, prefixCls + '-last-century-cell', isLast), 
              (0, _defineProperty3["default"])(_classNameMap, prefixCls + '-next-century-cell', isNext), 
              _classNameMap
            );
    
            var content = dStartDecade + '-' + dEndDecade;
            var clickHandler = void 0;
            if (isLast) {
              clickHandler = _this2.previousCentury;// 切换到上一个十年面板，即上一百年
            } else if (isNext) {
              clickHandler = _this2.nextCentury;// 切换到下一个十年面板，即下一百年
            } else {
              clickHandler = chooseDecade.bind(_this2, dStartDecade);
            }
    
            return _react2["default"].createElement(
              'td',
              {
                key: dStartDecade,
                onClick: clickHandler,
                role: 'gridcell',
                className: (0, _classnames2["default"])(classNameMap)
              },
              _react2["default"].createElement(
                'a',
                {
                  className: prefixCls + '-decade'
                },
                content
              )
            );
          });
          return _react2["default"].createElement(
            'tr',
            { key: decadeIndex, role: 'row' },
            tds
          );
        });
    
        return _react2["default"].createElement(
          'div',
          { className: this.prefixCls },
          _react2["default"].createElement(
            'div',
            { className: prefixCls + '-header' },
    
            // 上一百年切换按钮
            _react2["default"].createElement('a', {
              className: prefixCls + '-prev-century-btn',
              role: 'button',
              onClick: this.previousCentury,
              title: locale.previousCentury
            }),
    
            _react2["default"].createElement(
              'div',
              { className: prefixCls + '-century' },
              startYear,
              '-',
              endYear
            ),
    
            // 下一百年切换按钮
            _react2["default"].createElement('a', {
              className: prefixCls + '-next-century-btn',
              role: 'button',
              onClick: this.nextCentury,
              title: locale.nextCentury
            })
          ),
          _react2["default"].createElement(
            'div',
            { className: prefixCls + '-body' },
            _react2["default"].createElement(
              'table',
              { className: prefixCls + '-table', cellSpacing: '0', role: 'grid' },
              _react2["default"].createElement(
                'tbody',
                { className: prefixCls + '-tbody' },
                decadesEls
              )
            )
          )
        );
      };
    
      return DecadePanel;
    }(_react2["default"].Component);
    
    exports["default"] = DecadePanel;
    
    DecadePanel.propTypes = {
      locale: _react.PropTypes.object,
      value: _react.PropTypes.object,
      defaultValue: _react.PropTypes.object,
      rootPrefixCls: _react.PropTypes.string
    };
    
    DecadePanel.defaultProps = {
      onSelect: function onSelect() {}
    };
    
    // locale，语言包，决定文案
    // rootPrefixCls，样式类前缀
    // value，选中的年份，优先级高于props.defaultValue
    // defaultValue，选中的年份
    // onSelect，隐藏十年切换面板，显示年份选择面板；并更改年份选择面板选中值
    module.exports = exports['default'];



## MonthPanel

### 概述

渲染月份选择面板。 

### state属性 

* value，当前选中的日期对象
* showYearPanel，显示年份选择面板；1-显示，0-隐藏

### props属性

* locale，语言包，决定文案
* value, 当前选中的日期，影响选中的月份，优先级高于defaultValue
* defaultValue, 当前选中的日期，影响选中的月份
* rootPrefixCls: 样式类前缀
* onSelect，月份选择面板点选时执行函数
* onChange，年份选择面板点选时执行函数
* cellRender=(moment,locale)=>{}，渲染月份节点，优先级最高
* contentRender=(moment,locale)=>{}，渲染月份节点内容，优先级其次；最末以月份文案渲染月份节点

### 源码

    'use strict';
    
    Object.defineProperty(exports, "__esModule", {
      value: true
    });
    
    var _react = require('react');
    var _react2 = _interopRequireDefault(_react);
    
    var _YearPanel = require('../year/YearPanel');
    var _YearPanel2 = _interopRequireDefault(_YearPanel);
    
    var _MonthTable = require('./MonthTable');
    var _MonthTable2 = _interopRequireDefault(_MonthTable);
    
    function _interopRequireDefault(obj) { return obj && obj.__esModule ? obj : { "default": obj }; }
    
    // 跳转到上一年、下一年月份选择面板
    function goYear(direction) {
      var next = this.state.value.clone();
      next.add(direction, 'year');
      this.setAndChangeValue(next);
    }
    
    function noop() {}
    
    var MonthPanel = _react2["default"].createClass({
      displayName: 'MonthPanel',
    
      propTypes: {
        onChange: _react.PropTypes.func,
        disabledDate: _react.PropTypes.func,
        onSelect: _react.PropTypes.func
      },
    
      getDefaultProps: function getDefaultProps() {
        return {
          onChange: noop,
          onSelect: noop
        };
      },
      getInitialState: function getInitialState() {
        var props = this.props;
        this.nextYear = goYear.bind(this, 1);// 跳转到下一年月份选择面板
        this.previousYear = goYear.bind(this, -1);// 跳转到上一年月份选择面板
        this.prefixCls = props.rootPrefixCls + '-month-panel';
        return {
          value: props.value || props.defaultValue
        };
      },
      componentWillReceiveProps: function componentWillReceiveProps(nextProps) {
        if ('value' in nextProps) {
          this.setState({
            value: nextProps.value
          });
        }
      },
    
      // 年份选择面板，点击更改选中的年份，隐藏年份选择面板；执行props.onChange函数
      onYearPanelSelect: function onYearPanelSelect(current) {
        this.setState({
          showYearPanel: 0
        });
        this.setAndChangeValue(current);
      },
    
      // 年份选择面板，点击更改选中的年份；执行props.onChange函数
      setAndChangeValue: function setAndChangeValue(value) {
        this.setValue(value);
        this.props.onChange(value);
      },
    
      // 年份选择面板，点击更改选中的月份
      setAndSelectValue: function setAndSelectValue(value) {
        this.setValue(value);
        this.props.onSelect(value);
      },
    
      // 更改选中的年份或月份，并改变this.state.value
      setValue: function setValue(value) {
        if (!('value' in this.props)) {
          this.setState({
            value: value
          });
        }
      },
    
      // 显示年份选择面板
      showYearPanel: function showYearPanel() {
        this.setState({
          showYearPanel: 1
        });
      },
    
      render: function render() {
        var props = this.props;
        var value = this.state.value;
        var cellRender = props.cellRender;
        var contentRender = props.contentRender;
        var locale = props.locale;
        var year = value.year();
        var prefixCls = this.prefixCls;
    
        // 年份选择面板，this.state.showYearPanel为真时显示
        var yearPanel = void 0;
        if (this.state.showYearPanel) {
          yearPanel = _react2["default"].createElement(_YearPanel2["default"], {
            locale: locale,// 语言包，决定文案
            value: value,// 当前选中的日期，影响选中的年份
            rootPrefixCls: props.rootPrefixCls,// 样式类前缀
            onSelect: this.onYearPanelSelect// 年份选择面板点选时执行函数
          });
        }
    
        return _react2["default"].createElement(
          'div',
          { className: prefixCls, style: props.style },
          _react2["default"].createElement(
            'div',
            null,
            _react2["default"].createElement(
              'div',
    
              // 按钮，用于切换至上一年的月份选择面板
              { className: prefixCls + '-header' },
              _react2["default"].createElement('a', {
                className: prefixCls + '-prev-year-btn',
                role: 'button',
                onClick: this.previousYear,
                title: locale.previousYear
              }),
    
              _react2["default"].createElement(
                'a',
                {
                  className: prefixCls + '-year-select',
                  role: 'button',
                  onClick: this.showYearPanel,// 显示年份选择面板
                  title: locale.yearSelect
                },
                _react2["default"].createElement(
                  'span',
                  { className: prefixCls + '-year-select-content' },
                  year
                ),
                _react2["default"].createElement(
                  'span',
                  { className: prefixCls + '-year-select-arrow' },
                  'x'
                )
              ),
    
              // 按钮，用于切换至下一年的月份选择面板
              _react2["default"].createElement('a', {
                className: prefixCls + '-next-year-btn',
                role: 'button',
                onClick: this.nextYear,
                title: locale.nextYear
              })
            ),
            _react2["default"].createElement(
              'div',
              { className: prefixCls + '-body' },
    
              // 月份内容面板
              _react2["default"].createElement(_MonthTable2["default"], {
                disabledDate: props.disabledDate,// (moment)=>{}，不可选的日期
                onSelect: this.setAndSelectValue,// 月份选择面板点选时执行函数
                locale: locale,// 语言包，决定文案
                value: value,
                cellRender: cellRender,// (moment,locale)=>{}，渲染月份节点，优先级最高
                contentRender: contentRender,// (moment,locale)=>{}，渲染月份节点内容，优先级其次；最末以月份文案渲染月份节点
                prefixCls: prefixCls// 样式类前缀
              })
            )
          ),
          yearPanel
        );
      }
    });
    
    // locale,语言包，决定文案
    // value, 当前选中的日期，影响选中的月份，优先级高于defaultValue
    // defaultValue, 当前选中的日期，影响选中的月份
    // rootPrefixCls: 样式类前缀
    // onSelect，月份选择面板点选时执行函数
    // onChange，年份选择面板点选时执行函数
    // cellRender=(moment,locale)=>{}，渲染月份节点，优先级最高
    // contentRender=(moment,locale)=>{}，渲染月份节点内容，优先级其次；最末以月份文案渲染月份节点
    exports["default"] = MonthPanel;
    module.exports = exports['default'];



## MonthTable

### 概述

渲染月份选择面板内各月份节点内容。

### state属性

* value，当前选中的日期对象

### props属性

* prefixCls，样式类前缀
* disabledDate=(moment)=>{}，不可选的日期
* value，当前选中的日期
* cellRender=(moment,locale)=>{}，渲染月份节点，优先级最高
* contentRender=(moment,locale)=>{}，渲染月份节点内容，优先级其次；最末以月份文案渲染月份节点
* onSelect，月份选择面板点选时执行函数

### 源码

    'use strict';
    
    Object.defineProperty(exports, "__esModule", {
      value: true
    });
    
    var _defineProperty2 = require('babel-runtime/helpers/defineProperty');
    var _defineProperty3 = _interopRequireDefault(_defineProperty2);
    
    var _classCallCheck2 = require('babel-runtime/helpers/classCallCheck');
    var _classCallCheck3 = _interopRequireDefault(_classCallCheck2);
    
    var _possibleConstructorReturn2 = require('babel-runtime/helpers/possibleConstructorReturn');
    var _possibleConstructorReturn3 = _interopRequireDefault(_possibleConstructorReturn2);
    
    var _inherits2 = require('babel-runtime/helpers/inherits');
    var _inherits3 = _interopRequireDefault(_inherits2);
    
    var _react = require('react');
    var _react2 = _interopRequireDefault(_react);
    
    var _classnames = require('classnames');
    var _classnames2 = _interopRequireDefault(_classnames);
    
    var _index = require('../util/index');
    
    function _interopRequireDefault(obj) { return obj && obj.__esModule ? obj : { "default": obj }; }
    
    var ROW = 4;
    var COL = 3;
    
    // 月份选择面板点击月份，更改选中的月份
    function chooseMonth(month) {
      var next = this.state.value.clone();
      next.month(month);
      this.setAndSelectValue(next);
    }
    
    function noop() {}
    
    var MonthTable = function (_Component) {
      (0, _inherits3["default"])(MonthTable, _Component);
    
      function MonthTable(props) {
        (0, _classCallCheck3["default"])(this, MonthTable);
    
        var _this = (0, _possibleConstructorReturn3["default"])(this, _Component.call(this, props));
    
        _this.state = {
          value: props.value
        };
        return _this;
      }
    
      MonthTable.prototype.componentWillReceiveProps = function componentWillReceiveProps(nextProps) {
        if ('value' in nextProps) {
          this.setState({
            value: nextProps.value
          });
        }
      };
    
      // 更改选中的月份，并改变this.state.value
      MonthTable.prototype.setAndSelectValue = function setAndSelectValue(value) {
        this.setState({
          value: value
        });
        this.props.onSelect(value);
      };
    
      // 月份节点数据，显示12个年
      MonthTable.prototype.months = function months() {
        var value = this.state.value;
        var current = value.clone();
        var months = [];
        var localeData = value.localeData();
        var index = 0;
        for (var rowIndex = 0; rowIndex < ROW; rowIndex++) {
          months[rowIndex] = [];
          for (var colIndex = 0; colIndex < COL; colIndex++) {
            current.month(index);
            var content = localeData.monthsShort(current);
            months[rowIndex][colIndex] = {
              value: index,
              content: content,
              title: content
            };
            index++;
          }
        }
        return months;
      };
    
      MonthTable.prototype.render = function render() {
        var _this2 = this;
    
        var props = this.props;
        var value = this.state.value;
        var today = (0, _index.getTodayTime)(value);
        var months = this.months();
        var currentMonth = value.month();
        var prefixCls = props.prefixCls,
            locale = props.locale,
            contentRender = props.contentRender,
            cellRender = props.cellRender;
    
        var monthsEls = months.map(function (month, index) {
          var tds = month.map(function (monthData) {
            var _classNameMap;
    
            // 月份节点是否不可选
            var disabled = false;
            if (props.disabledDate) {
              var testValue = value.clone();
              testValue.month(monthData.value);
              disabled = props.disabledDate(testValue);
            }
    
            // 月份节点的样式类
            var classNameMap = (
              _classNameMap = {}, 
              (0, _defineProperty3["default"])(_classNameMap, prefixCls + '-cell', 1), 
              (0, _defineProperty3["default"])(_classNameMap, prefixCls + '-cell-disabled', disabled), 
              (0, _defineProperty3["default"])(_classNameMap, prefixCls + '-selected-cell', monthData.value === currentMonth), 
              (0, _defineProperty3["default"])(_classNameMap, prefixCls + '-current-cell', today.year() === value.year() && monthData.value === today.month()), 
              _classNameMap
            );
    
            // 渲染月份节点
            var cellEl = void 0;
            if (cellRender) {
              var currentValue = value.clone();
              currentValue.month(monthData.value);
              cellEl = cellRender(currentValue, locale);
            } else {
              var content = void 0;
              if (contentRender) {
                var _currentValue = value.clone();
                _currentValue.month(monthData.value);
                content = contentRender(_currentValue, locale);
              } else {
                content = monthData.content;
              }
              cellEl = _react2["default"].createElement(
                'a',
                { className: prefixCls + '-month' },
                content
              );
            }
    
            return _react2["default"].createElement(
              'td',
              {
                role: 'gridcell',
                key: monthData.value,
                onClick: disabled ? null : chooseMonth.bind(_this2, monthData.value),// 月份选择面板点击月份，更改选中的月份
                title: monthData.title,
                className: (0, _classnames2["default"])(classNameMap)
              },
              cellEl
            );
          });
          return _react2["default"].createElement(
            'tr',
            { key: index, role: 'row' },
            tds
          );
        });
    
        return _react2["default"].createElement(
          'table',
          { className: prefixCls + '-table', cellSpacing: '0', role: 'grid' },
          _react2["default"].createElement(
            'tbody',
            { className: prefixCls + '-tbody' },
            monthsEls
          )
        );
      };
    
      return MonthTable;
    }(_react.Component);
    
    MonthTable.defaultProps = {
      onSelect: noop
    };
    
    MonthTable.propTypes = {
      onSelect: _react.PropTypes.func,
      cellRender: _react.PropTypes.func,
      prefixCls: _react.PropTypes.string,
      value: _react.PropTypes.object
    };
    
    // prefixCls，样式类前缀
    // disabledDate=(moment)=>{}，不可选的日期
    // value，当前选中的日期
    // cellRender=(moment,locale)=>{}，渲染月份节点，优先级最高
    // contentRender=(moment,locale)=>{}，渲染月份节点内容，优先级其次；最末以月份文案渲染月份节点
    // onSelect，月份选择面板点选时执行函数
    exports["default"] = MonthTable;
    module.exports = exports['default'];
