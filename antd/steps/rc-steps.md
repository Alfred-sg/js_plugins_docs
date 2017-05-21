# rc-steps 2.2.4

## 接口

rc-steps组件可接受的props属性：
* current，指定当前步骤，从0开始记数。在子Step元素中，可以通过status属性覆盖状态
* prefixCls，样式前缀，默认为'rc-steps'
* iconPrefix，图标样式
* direction，指定步骤条方向，可接受"horizontal"、"vertical"；默认为'horizontal'
* labelPlacement，默认为'horizontal'
* status，指定当前步骤的状态，可接受"wait"、"process"、"finish"、"error"；默认为"process"
* size，指定大小，可接受"default"、"small"
* className，样式
* style，样式，渲染Steps节点，特别取其background、backgroundColor属性传入Step组件
* children，子节点Step组件，劫持渲染

rc-step组件作为Steps子组件时可接受的props属性：
* className，样式
* style，样式，Steps组件的样式，取其background、backgroundColor属性传入Step组件
* status，step节点状态"wait"、"process"、"finish"、"error"；优先级高于Steps组件的current属性
* icon，图标节点，可接受字符串或react节点
* title，step节点的标题
* description，step节点的描述

rc-step组件中由由Steps注入props属性：
* prefixCls，样式前缀，默认为'rc-steps'
* tailWidth，step节点的宽度，水平展开取百分比；否则取null
* iconPrefix，图标样式前缀
* adjustMarginRight，step节点偏移量，赋给marginRight属性
* stepLast，是否最后一个节点
* stepNumber，字符串形式的step节点序号

## 源码

### index.js 

    'use strict';
    
    var Steps = require('./Steps');
    Steps.Step = require('./Step');
    
    module.exports = Steps;

### Steps.js 

    'use strict';
    
    Object.defineProperty(exports, "__esModule", {
      value: true
    });
        
    // 浅拷贝    
    var _extends = Object.assign || function (target) {    
      for (var i = 1; i < arguments.length; i++) {     
        var source = arguments[i];     
        for (var key in source) {     
          if (Object.prototype.hasOwnProperty.call(source, key)) {     
            target[key] = source[key];     
          }     
        }     
      }     
      return target;     
    };    
       
    var _react = require('react');
    var _react2 = _interopRequireDefault(_react);
    
    var _reactDom = require('react-dom');
    var _reactDom2 = _interopRequireDefault(_reactDom);
    
    var _classnames = require('classnames');
    var _classnames2 = _interopRequireDefault(_classnames);
    
    function _interopRequireDefault(obj) { return obj && obj.__esModule ? obj : { "default": obj }; }
    
    // 以defaults设置obj对象某属性key的默认值，当obj[key]为undefined时触发  
    function _defaults(obj, defaults) {   
      var keys = Object.getOwnPropertyNames(defaults);   
      for (var i = 0; i < keys.length; i++) {   
        var key = keys[i];   
        var value = Object.getOwnPropertyDescriptor(defaults, key);   
        if (value && value.configurable && obj[key] === undefined) {   
          Object.defineProperty(obj, key, value);   
        }   
      }   
      return obj;   
    }
    
    // obj对象添加方法或属性，可能采用属性描述符方式  
    function _defineProperty(obj, key, value) {    
      if (key in obj) {    
        Object.defineProperty(obj, key,     
          { value: value, enumerable: true, configurable: true, writable: true });     
      } else {     
        obj[key] = value;     
      }     
      return obj;     
    }
    
    // 浅拷贝obj对象，但不包含在keys中的属性与方法  
    function _objectWithoutProperties(obj, keys) {   
      var target = {};   
      for (var i in obj) {   
          if (keys.indexOf(i) >= 0) continue;   
          if (!Object.prototype.hasOwnProperty.call(obj, i)) continue;   
          target[i] = obj[i];   
      }   
      return target;   
    } 
       
    // 构造函数只能使用new关键字调用    
    function _classCallCheck(instance, Constructor) {     
      if (!(instance instanceof Constructor)) { throw new TypeError("Cannot call a class as a function"); }     
    }
       
    function _possibleConstructorReturn(self, call) {    
      // 构造函数未曾实例化，ReferenceError引用错误 Uncaught ReferenceError:XXXX is not defined    
      if (!self) { throw new ReferenceError("this hasn't been initialised - super() hasn't been called"); }     
      return call && (typeof call === "object" || typeof call === "function") ? call : self;     
    }    
       
    // 继承，设置subClass.prototype为{constuctor:subClass}，{constuctor:subClass}的原型是superClass.prototype    
    // subClass.__proto__为superClass    
    function _inherits(subClass, superClass) {    
      if (typeof superClass !== "function" && superClass !== null) {    
        throw new TypeError("Super expression must either be null or a function, not " + typeof superClass);     
      }    
      
      // 实例可访问原型属性或方法，构造函数不能    
      // Object.create创建对象，首参为该对象的原型prototype，次参为对象的自有属性，并设置属性描述符    
      subClass.prototype = Object.create(superClass && superClass.prototype,     
        { constructor: { value: subClass, enumerable: false, writable: true, configurable: true }     
      });     
      
      // Object.setPrototypeOf(subClass, superClass)，设置subClass.__proto__为superClass    
      // Object.getPrototypeOf(subClass)，获取subClass.__proto__    
      // 构造函数可访问，实例不能    
      if (superClass) Object.setPrototypeOf ?     
        Object.setPrototypeOf(subClass, superClass) : subClass.__proto__ = superClass;     
    }    
     
    var Steps = function (_React$Component) {
      _inherits(Steps, _React$Component);
    
      function Steps(props) {
        _classCallCheck(this, Steps);
    
        var _this = _possibleConstructorReturn(this, _React$Component.call(this, props));
    
        // 计算最后一个Step节点的宽度，存入this.state.lastStepOffsetWidth
        // 作为其余Step节点的marginRight偏移量，Step节点宽度取百分比
        _this.calcLastStepOffsetWidth = function () {
          var domNode = _reactDom2["default"].findDOMNode(_this);
          if (domNode.children.length > 0) {
            if (_this.calcTimeout) {
              clearTimeout(_this.calcTimeout);
            }
            _this.calcTimeout = setTimeout(function () {
              // +1 for fit edge bug of digit width, like 35.4px
              var lastStepOffsetWidth = (domNode.lastChild.offsetWidth || 0) + 1;
              if (_this.state.lastStepOffsetWidth === lastStepOffsetWidth) {
                return;
              }
              _this.setState({ lastStepOffsetWidth: lastStepOffsetWidth });
            });
          }
        };
    
        _this.state = {
          lastStepOffsetWidth: 0
        };
        return _this;
      }
    
      Steps.prototype.componentDidMount = function componentDidMount() {
        this.calcLastStepOffsetWidth();
      };
    
      Steps.prototype.componentDidUpdate = function componentDidUpdate() {
        this.calcLastStepOffsetWidth();
      };
    
      Steps.prototype.componentWillUnmount = function componentWillUnmount() {
        if (this.calcTimeout) {
          clearTimeout(this.calcTimeout);
        }
      };
    
      Steps.prototype.render = function render() {
        var _classNames,
            _this2 = this;
    
        var props = this.props;
        var prefixCls = props.prefixCls;
        var _props$style = props.style;
        var style = _props$style === undefined ? {} : _props$style;
        var className = props.className;
        var children = props.children;
        var direction = props.direction;
        var labelPlacement = props.labelPlacement;
        var iconPrefix = props.iconPrefix;
        var status = props.status;
        var size = props.size;
        var current = props.current;
    
        var restProps = _objectWithoutProperties(props, ['prefixCls', 'style', 'className', 'children', 
          'direction', 'labelPlacement', 'iconPrefix', 'status', 'size', 'current']);
    
        var lastIndex = children.length - 1;
        // 是否计算step节点的marginRight偏移量，尚未计算隐藏Steps组件、且不设置step节点宽度
        var reLayouted = this.state.lastStepOffsetWidth > 0;
        var classString = (0, _classnames2["default"])((
          _classNames = {}, 
          _defineProperty(_classNames, prefixCls, true), 
          _defineProperty(_classNames, prefixCls + '-' + size, size), 
          _defineProperty(_classNames, prefixCls + '-' + direction, true), 
          _defineProperty(_classNames, prefixCls + '-label-' + labelPlacement, direction === 'horizontal'), 
          _defineProperty(_classNames, prefixCls + '-hidden', !reLayouted), 
          _defineProperty(_classNames, className, className), 
          _classNames
        ));
    
        return _react2["default"].createElement(
          'div',
          _extends({ className: classString, style: style }, restProps),
          _react2["default"].Children.map(children, function (ele, idx) {
            var tailWidth = direction === 'vertical' || idx === lastIndex || !reLayouted ? 
              null : 100 / lastIndex + '%';
            var adjustMarginRight = direction === 'vertical' || idx === lastIndex ? 
              null : -Math.round(_this2.state.lastStepOffsetWidth / lastIndex + 1);
            var np = {
              stepNumber: (idx + 1).toString(),
              stepLast: idx === lastIndex,
              tailWidth: tailWidth,
              adjustMarginRight: adjustMarginRight,
              prefixCls: prefixCls,
              iconPrefix: iconPrefix,
              wrapperStyle: style
            };
    
            if (status === 'error' && idx === current - 1) {
              np.className = props.prefixCls + '-next-error';
            }
    
            if (!ele.props.status) {
              if (idx === current) {
                np.status = status;
              } else if (idx < current) {
                np.status = 'finish';
              } else {
                np.status = 'wait';
              }
            }
    
            // React.cloneElement(ele,np)时会复制节点原有的props，再添加或替换ele组件的defaultProps或np
            return _react2["default"].cloneElement(ele, np);
          }, this)
        );
      };
    
      return Steps;
    }(_react2["default"].Component);
    
    exports["default"] = Steps;
    
    // current，指定当前步骤，从0开始记数。在子Step元素中，可以通过status属性覆盖状态
    // prefixCls，样式前缀，默认为'rc-steps'
    // iconPrefix，图标样式
    // direction，指定步骤条方向，可接受"horizontal"、"vertical"；默认为'horizontal'
    // labelPlacement，默认为'horizontal'
    // status，指定当前步骤的状态，可接受"wait"、"process"、"finish"、"error"；默认为"process"
    // size，指定大小，可接受"default"、"small"
    // style，样式，渲染Steps节点，特别取其background、backgroundColor属性传入Step组件
    // children，子节点Step组件，劫持渲染
    Steps.propTypes = {
      prefixCls: _react.PropTypes.string,
      iconPrefix: _react.PropTypes.string,
      direction: _react.PropTypes.string,
      labelPlacement: _react.PropTypes.string,
      children: _react.PropTypes.any,
      status: _react.PropTypes.string,
      size: _react.PropTypes.string
    };
    
    Steps.defaultProps = {
      prefixCls: 'rc-steps',
      iconPrefix: 'rc',
      direction: 'horizontal',
      labelPlacement: 'horizontal',
      current: 0,
      status: 'process',
      size: ''
    };
    
    module.exports = exports['default'];

### Step.js 

    'use strict';
    
    Object.defineProperty(exports, "__esModule", {
      value: true
    });
        
    // 浅拷贝    
    var _extends = Object.assign || function (target) {    
      for (var i = 1; i < arguments.length; i++) {     
        var source = arguments[i];     
        for (var key in source) {     
          if (Object.prototype.hasOwnProperty.call(source, key)) {     
            target[key] = source[key];     
          }     
        }     
      }     
      return target;     
    };    
      
    var _react = require('react');
    var _react2 = _interopRequireDefault(_react);
    
    var _classnames = require('classnames');
    var _classnames2 = _interopRequireDefault(_classnames);
    
    function _interopRequireDefault(obj) { return obj && obj.__esModule ? obj : { "default": obj }; }
    
    // 以defaults设置obj对象某属性key的默认值，当obj[key]为undefined时触发  
    function _defaults(obj, defaults) {   
      var keys = Object.getOwnPropertyNames(defaults);   
      for (var i = 0; i < keys.length; i++) {   
        var key = keys[i];   
        var value = Object.getOwnPropertyDescriptor(defaults, key);   
        if (value && value.configurable && obj[key] === undefined) {   
          Object.defineProperty(obj, key, value);   
        }   
      }   
      return obj;   
    }
    
    // obj对象添加方法或属性，可能采用属性描述符方式  
    function _defineProperty(obj, key, value) {    
      if (key in obj) {    
        Object.defineProperty(obj, key,     
          { value: value, enumerable: true, configurable: true, writable: true });     
      } else {     
        obj[key] = value;     
      }     
      return obj;     
    }
    
    // 浅拷贝obj对象，但不包含在keys中的属性与方法  
    function _objectWithoutProperties(obj, keys) {   
      var target = {};   
      for (var i in obj) {   
          if (keys.indexOf(i) >= 0) continue;   
          if (!Object.prototype.hasOwnProperty.call(obj, i)) continue;   
          target[i] = obj[i];   
      }   
      return target;   
    } 
       
    // 构造函数只能使用new关键字调用    
    function _classCallCheck(instance, Constructor) {     
      if (!(instance instanceof Constructor)) { throw new TypeError("Cannot call a class as a function"); }     
    }
       
    function _possibleConstructorReturn(self, call) {    
      // 构造函数未曾实例化，ReferenceError引用错误 Uncaught ReferenceError:XXXX is not defined    
      if (!self) { throw new ReferenceError("this hasn't been initialised - super() hasn't been called"); }     
      return call && (typeof call === "object" || typeof call === "function") ? call : self;     
    }    
       
    // 继承，设置subClass.prototype为{constuctor:subClass}，{constuctor:subClass}的原型是superClass.prototype    
    // subClass.__proto__为superClass    
    function _inherits(subClass, superClass) {    
      if (typeof superClass !== "function" && superClass !== null) {    
        throw new TypeError("Super expression must either be null or a function, not " + typeof superClass);     
      }    
      
      // 实例可访问原型属性或方法，构造函数不能    
      // Object.create创建对象，首参为该对象的原型prototype，次参为对象的自有属性，并设置属性描述符    
      subClass.prototype = Object.create(superClass && superClass.prototype,     
        { constructor: { value: subClass, enumerable: false, writable: true, configurable: true }     
      });     
      
      // Object.setPrototypeOf(subClass, superClass)，设置subClass.__proto__为superClass    
      // Object.getPrototypeOf(subClass)，获取subClass.__proto__    
      // 构造函数可访问，实例不能    
      if (superClass) Object.setPrototypeOf ?     
        Object.setPrototypeOf(subClass, superClass) : subClass.__proto__ = superClass;     
    }    
    
    function isString(str) {
      return typeof str === 'string';
    }
    
    var Step = function (_React$Component) {
      _inherits(Step, _React$Component);
    
      function Step() {
        _classCallCheck(this, Step);
    
        return _possibleConstructorReturn(this, _React$Component.apply(this, arguments));
      }
    
      Step.prototype.render = function render() {
        var _classNames, _classNames2;
    
        var _props = this.props;
        var className = _props.className;
        var prefixCls = _props.prefixCls;
        var style = _props.style;
        var tailWidth = _props.tailWidth;
        var _props$status = _props.status;
        var status = _props$status === undefined ? 'wait' : _props$status;
        var iconPrefix = _props.iconPrefix;
        var icon = _props.icon;
        var wrapperStyle = _props.wrapperStyle;
        var adjustMarginRight = _props.adjustMarginRight;
        var stepLast = _props.stepLast;
        var stepNumber = _props.stepNumber;
        var description = _props.description;
        var title = _props.title;
    
        var restProps = _objectWithoutProperties(_props, ['className', 'prefixCls', 'style', 'tailWidth', 
          'status', 'iconPrefix', 'icon', 'wrapperStyle', 'adjustMarginRight', 'stepLast', 'stepNumber', 
          'description', 'title']);
    
        var iconClassName = (0, _classnames2["default"])((
          _classNames = {}, 
          _defineProperty(_classNames, prefixCls + '-icon', true), 
          _defineProperty(_classNames, iconPrefix + 'icon', true), 
          _defineProperty(_classNames, iconPrefix + 'icon-' + icon, icon && isString(icon)), 
          _defineProperty(_classNames, iconPrefix + 'icon-check', !icon && status === 'finish'), 
          _defineProperty(_classNames, iconPrefix + 'icon-cross', !icon && status === 'error'), 
          _classNames
        ));
    
        var iconNode = void 0;
        if (icon && !isString(icon)) {
          iconNode = _react2["default"].createElement(
            'span',
            { className: prefixCls + '-icon' },
            icon
          );
        } else if (icon || status === 'finish' || status === 'error') {
          iconNode = _react2["default"].createElement('span', { className: iconClassName });
        } else {
          iconNode = _react2["default"].createElement(
            'span',
            { className: prefixCls + '-icon' },
            stepNumber
          );
        }
    
        var classString = (0, _classnames2["default"])((
          _classNames2 = {}, 
          _defineProperty(_classNames2, prefixCls + '-item', true), 
          _defineProperty(_classNames2, prefixCls + '-item-last', stepLast), 
          _defineProperty(_classNames2, prefixCls + '-status-' + status, true), 
          _defineProperty(_classNames2, prefixCls + '-custom', icon), 
          _defineProperty(_classNames2, className, !!className), 
          _classNames2
        ));
    
        return _react2["default"].createElement(
          'div',
          _extends({}, restProps, {
            className: classString,
            style: _extends({ width: tailWidth, marginRight: adjustMarginRight }, style)
          }),
    
          // 绘制水平竖线，绝对定位，z坐标上较图标、标题、描述节点靠后
          stepLast ? '' : _react2["default"].createElement(
            'div',
            { ref: 'tail', className: prefixCls + '-tail' },
            _react2["default"].createElement('i', null)
          ),
    
          // 含图标节点、标题、描述节点
          _react2["default"].createElement(
            'div',
            { className: prefixCls + '-step' },
            _react2["default"].createElement(
              'div',
              {
                className: prefixCls + '-head',
                style: { background: wrapperStyle.background || wrapperStyle.backgroundColor }
              },
              _react2["default"].createElement(
                'div',
                { className: prefixCls + '-head-inner' },
                iconNode
              )
            ),
            _react2["default"].createElement(
              'div',
              { ref: 'main', className: prefixCls + '-main' },
              _react2["default"].createElement(
                'div',
                {
                  className: prefixCls + '-title',
                  style: { background: wrapperStyle.background || wrapperStyle.backgroundColor }
                },
                title
              ),
              description ? _react2["default"].createElement(
                'div',
                { className: prefixCls + '-description' },
                description
              ) : ''
            )
          )
        );
      };
    
      return Step;
    }(_react2["default"].Component);
    
    exports["default"] = Step;
    
    // className，样式
    // prefixCls，样式前缀，默认为'rc-steps'
    // style，样式，Steps组件的样式，取其background、backgroundColor属性传入Step组件
    // tailWidth，step节点的宽度，水平展开取百分比；否则取null
    // status，step节点状态"wait"、"process"、"finish"、"error"；优先级高于Steps组件的current属性
    // iconPrefix，图标样式前缀
    // icon，图标节点，可接受字符串或react节点
    // adjustMarginRight，step节点偏移量，赋给marginRight属性
    // stepLast，是否最后一个节点
    // stepNumber，字符串形式的step节点序号
    // title，step节点的标题
    // description，step节点的描述
    Step.propTypes = {
      className: _react.PropTypes.string,
      prefixCls: _react.PropTypes.string,
      style: _react.PropTypes.object,
      wrapperStyle: _react.PropTypes.object,
      tailWidth: _react.PropTypes.oneOfType([_react.PropTypes.number, _react.PropTypes.string]),
      status: _react.PropTypes.string,
      iconPrefix: _react.PropTypes.string,
      icon: _react.PropTypes.node,
      adjustMarginRight: _react.PropTypes.oneOfType([_react.PropTypes.number, _react.PropTypes.string]),
      stepLast: _react.PropTypes.bool,
      stepNumber: _react.PropTypes.string,
      description: _react.PropTypes.any,
      title: _react.PropTypes.any
    };
    
    module.exports = Step;
    module.exports = exports['default'];