# Picker、placements

Picker组件将各式日期选择组件通过rc-trigger机制展示为浮层；placements设定展示的位置，如左上等。

## Picker

### 概述

将各式日期选择组件通过rc-trigger机制展示为浮层。

### props属性

* prefixCls样式类前缀
* style，样式
* align，浮层位置调整数据
* placement，浮层位置调整数据
* animation，浮层显隐时添加的css动效
* transitionName，浮层显隐时添加的css动效
* disabled，日期选择组件是否可用，通过"rc-trigger"限制输入框点击不弹出浮层
* calendar，浮层内部的日期选择组件
*   由getCalendarContainer方法传入ref、defaultValue、selectedValue、onKeyDown、onOk、onSelect、onClear属性
*   ref=(calendar)=>{}方法，使this.calendarInstance=calendar引用日期选择组件
*   defaultValue属性，默认初始化时间，this.state.value优先级高于calendar.props.defaultValue
*   selectedValue=this.state.value属性，默认选中时间
*   onKeyDown=this.onCalendarKeyDown方法，键盘esc按键执行函数
*   onOk=this.onCalendarOk方法，键盘确认按键执行函数
*   onSelect=this.onCalendarSelect方法，日期选择组件点选时执行函数
*   onClear=this.onCalendarClear方法，点击取消按键时执行函数
* getCalendarContainer，劫持渲染浮层内部的日期选择组件
* onOpenChange(isOpen)，浮层显示隐藏时执行函数
* onChange(value)，浮层内日期选择组件点选时触发执行函数
* children=(state,props)=>{}，以函数形式设置浮层内部节点
* open、defaultOpen，浮层初始化展开状态，open优先级高于defaultOpen
* value、defaultValue，初始化时间

### 源码

    'use strict';
    
    Object.defineProperty(exports, "__esModule", {
      value: true
    });
    
    var _react = require('react');
    var _react2 = _interopRequireDefault(_react);
    
    var _reactDom = require('react-dom');
    var _reactDom2 = _interopRequireDefault(_reactDom);
    
    var _createChainedFunction = require('rc-util/lib/createChainedFunction');
    var _createChainedFunction2 = _interopRequireDefault(_createChainedFunction);
    
    var _KeyCode = require('rc-util/lib/KeyCode');
    var _KeyCode2 = _interopRequireDefault(_KeyCode);
    
    var _placements = require('./picker/placements');
    var _placements2 = _interopRequireDefault(_placements);
    
    var _rcTrigger = require('rc-trigger');
    var _rcTrigger2 = _interopRequireDefault(_rcTrigger);
    
    function _interopRequireDefault(obj) { return obj && obj.__esModule ? obj : { "default": obj }; }
    
    function noop() {}
    
    function refFn(field, component) {
      this[field] = component;
    }
    
    var Picker = _react2["default"].createClass({
      displayName: 'Picker',
    
      propTypes: {
        animation: _react.PropTypes.oneOfType([_react.PropTypes.func, _react.PropTypes.string]),
        disabled: _react.PropTypes.bool,
        transitionName: _react.PropTypes.string,
        onChange: _react.PropTypes.func,
        onOpenChange: _react.PropTypes.func,
        children: _react.PropTypes.func,
        getCalendarContainer: _react.PropTypes.func,
        calendar: _react.PropTypes.element,
        style: _react.PropTypes.object,
        open: _react.PropTypes.bool,
        defaultOpen: _react.PropTypes.bool,
        prefixCls: _react.PropTypes.string,
        placement: _react.PropTypes.any,
        value: _react.PropTypes.oneOfType([_react.PropTypes.object, _react.PropTypes.array]),
        defaultValue: _react.PropTypes.oneOfType([_react.PropTypes.object, _react.PropTypes.array]),
        align: _react.PropTypes.object
      },
    
      getDefaultProps: function getDefaultProps() {
        return {
          prefixCls: 'rc-calendar-picker',
          style: {},
          align: {},
          placement: 'bottomLeft',
          defaultOpen: false,
          onChange: noop,
          onOpenChange: noop
        };
      },
      getInitialState: function getInitialState() {
        var props = this.props;
        var open = void 0;
        if ('open' in props) {
          open = props.open;
        } else {
          open = props.defaultOpen;
        }
        var value = props.value || props.defaultValue;
    
        // refs引用函数(component)=>{}，设置this.calendarInstance=component日期选择组件
        this.saveCalendarRef = refFn.bind(this, 'calendarInstance');
        return {
          open: open,
          value: value
        };
      },
      componentWillReceiveProps: function componentWillReceiveProps(nextProps) {
        var value = nextProps.value,
            open = nextProps.open;
    
        if ('value' in nextProps) {
          this.setState({
            value: value
          });
        }
        if (open !== undefined) {
          this.setState({
            open: open
          });
        }
      },
      // 日期选择组件显示状态时，按esc键隐藏日期选择组件，picker组件获得焦点
      onCalendarKeyDown: function onCalendarKeyDown(event) {
        if (event.keyCode === _KeyCode2["default"].ESC) {
          event.stopPropagation();
          this.close(this.focus);
        }
      },
      // 日期选择组件calendar点选时执行函数，更改this.state.value，按data.source及时间面板类型显示或隐藏浮层
      // 执行props.onChange方法，改变输入框的时间值
      onCalendarSelect: function onCalendarSelect(value) {
        var cause = arguments.length > 1 && arguments[1] !== undefined ? arguments[1] : {};
    
        var props = this.props;
        if (!('value' in props)) {
          this.setState({
            value: value
          });
        }
        if (cause.source === 'keyboard' || !props.calendar.props.timePicker && 
          cause.source !== 'dateInput' || cause.source === 'todayButton') {
          this.close(this.focus);
        }
        props.onChange(value);
      },
      // 日期选择组件隐藏时，点击下箭头，显示日期选择组件，并使日期选择组件获得焦点
      onKeyDown: function onKeyDown(event) {
        if (event.keyCode === _KeyCode2["default"].DOWN && !this.state.open) {
          this.open(this.focusCalendar);
          event.preventDefault();
        }
      },
      // 日期选择组件点击确认按钮时，隐藏日期选择组件，picker组件获得焦点
      onCalendarOk: function onCalendarOk() {
        this.close(this.focus);
      },
      // 日期选择组件点击清空按钮时，隐藏日期选择组件，picker组件获得焦点
      onCalendarClear: function onCalendarClear() {
        this.close(this.focus);
      },
      // 显示日期选择组件this.calendarInstance，并使日期选择组件获得焦点
      onVisibleChange: function onVisibleChange(open) {
        this.setOpen(open, this.focusCalendar);
      },
      // 劫持渲染props.calendar，props改变ref、defaultValue、selectedValue、onKeyDown、onOk、onSelect、onClear属性
      getCalendarElement: function getCalendarElement() {
        var props = this.props;
        var state = this.state;
        var calendarProps = props.calendar.props;
        var value = state.value;
    
        var defaultValue = void 0;
        if (Array.isArray(value)) {
          defaultValue = value[0];
        } else {
          defaultValue = value;
        }
        var extraProps = {
          ref: this.saveCalendarRef,// 设置this.calendarInstance=component日期选择组件
          defaultValue: defaultValue || calendarProps.defaultValue,
          selectedValue: value,
          onKeyDown: this.onCalendarKeyDown,
          onOk: (0, _createChainedFunction2["default"])(calendarProps.onOk, this.onCalendarOk),
          onSelect: (0, _createChainedFunction2["default"])(calendarProps.onSelect, this.onCalendarSelect),
          onClear: (0, _createChainedFunction2["default"])(calendarProps.onClear, this.onCalendarClear)
        };
    
        return _react2["default"].cloneElement(props.calendar, extraProps);
      },
      // 日期选择浮层改变显示状态this.state.open时，执行callback回调，及onOpenChange(isOpen)回调
      setOpen: function setOpen(open, callback) {
        var onOpenChange = this.props.onOpenChange;
    
        if (this.state.open !== open) {
          if (!('open' in this.props)) {
            this.setState({
              open: open
            }, callback);
          }
          onOpenChange(open);
        }
      },
      // 显示日期选择浮层，执行callback回调
      open: function open(callback) {
        this.setOpen(true, callback);
      },
      // 隐藏日期选择浮层，执行callback回调
      close: function close(callback) {
        this.setOpen(false, callback);
      },
      // picker节点获得焦点
      focus: function focus() {
        if (!this.state.open) {
          _reactDom2["default"].findDOMNode(this).focus();
        }
      },
      // 日期选择组件Calender获得焦点
      focusCalendar: function focusCalendar() {
        if (this.state.open && this.calendarInstance !== null) {
          this.calendarInstance.focus();
        }
      },
      render: function render() {
        var props = this.props;
        var prefixCls = props.prefixCls,
            placement = props.placement,
            style = props.style,
            getCalendarContainer = props.getCalendarContainer,
            align = props.align,
            animation = props.animation,
            disabled = props.disabled,
            transitionName = props.transitionName,
            children = props.children;
    
        var state = this.state;
        return _react2["default"].createElement(
          _rcTrigger2["default"],
          {
            popup: this.getCalendarElement(),// 浮层节点
            popupAlign: align,// 浮层位置调整数据
            builtinPlacements: _placements2["default"],// 浮层位置调整数据
            popupPlacement: placement,// 浮层位置调整数据
            action: disabled && !state.open ? [] : ['click'],// 触发浮层显示的方法
            destroyPopupOnHide: true,// 浮层隐藏时，是否卸载该浮层元素
            getPopupContainer: getCalendarContainer,// 设置浮层的容器，默认的容器document.body
            popupStyle: style,// 浮层元素的样式
            popupAnimation: animation,// 浮层显隐时添加的css动效
            popupTransitionName: transitionName,// 浮层显隐时添加的css动效
            popupVisible: state.open,// 挂载时设置浮层是否显示
            onPopupVisibleChange: this.onVisibleChange,// 浮层显示隐藏时执行的回调函数
            prefixCls: prefixCls// 样式类前缀
          },
          _react2["default"].cloneElement(children(state, props), { onKeyDown: this.onKeyDown })
        );
      }
    });
    
    // prefixCls样式类前缀
    // style，样式
    // align，浮层位置调整数据
    // placement，浮层位置调整数据
    // animation，浮层显隐时添加的css动效
    // transitionName，浮层显隐时添加的css动效
    // disabled，日期选择组件是否可用，通过"rc-trigger"限制输入框点击不弹出浮层
    // calendar，浮层内部的日期选择组件
    //    由getCalendarContainer方法传入ref、defaultValue、selectedValue、onKeyDown、onOk、onSelect、onClear属性
    //    ref=(calendar)=>{}方法，使this.calendarInstance=calendar引用日期选择组件
    //    defaultValue属性，默认初始化时间，this.state.value优先级高于calendar.props.defaultValue
    //    selectedValue=this.state.value属性，默认选中时间
    //    onKeyDown=this.onCalendarKeyDown方法，键盘esc按键执行函数
    //    onOk=this.onCalendarOk方法，键盘确认按键执行函数
    //    onSelect=this.onCalendarSelect方法，日期选择组件点选时执行函数
    //    onClear=this.onCalendarClear方法，点击取消按键时执行函数
    // getCalendarContainer，劫持渲染浮层内部的日期选择组件
    // onOpenChange(isOpen)，浮层显示隐藏时执行函数
    // onChange(value)，浮层内日期选择组件点选时触发执行函数
    // children=(state,props)=>{}，以函数形式设置浮层内部节点
    // open、defaultOpen，浮层初始化展开状态，open优先级高于defaultOpen
    // value、defaultValue，初始化时间
    exports["default"] = Picker;
    module.exports = exports['default'];



## placements

### 概述

用于作为浮层的显示位置属性。

### 源码

    'use strict';
    
    Object.defineProperty(exports, "__esModule", {
      value: true
    });
    var autoAdjustOverflow = {
      adjustX: 1,
      adjustY: 1
    };
    
    var targetOffset = [0, 0];
    
    var placements = {
      bottomLeft: {
        points: ['tl', 'tl'],
        overflow: autoAdjustOverflow,
        offset: [0, -3],
        targetOffset: targetOffset
      },
      bottomRight: {
        points: ['tr', 'tr'],
        overflow: autoAdjustOverflow,
        offset: [0, -3],
        targetOffset: targetOffset
      },
      topRight: {
        points: ['br', 'br'],
        overflow: autoAdjustOverflow,
        offset: [0, 3],
        targetOffset: targetOffset
      },
      topLeft: {
        points: ['bl', 'bl'],
        overflow: autoAdjustOverflow,
        offset: [0, 3],
        targetOffset: targetOffset
      }
    };
    
    exports["default"] = placements;
    module.exports = exports['default'];