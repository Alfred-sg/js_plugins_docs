# steps 2.6.3

## 接口

Steps组件可接受的props属性：
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

Step组件作为Steps子组件时可接受的props属性，其余属性由Steps注入：
* className，样式
* style，样式，Steps组件的样式，取其background、backgroundColor属性传入Step组件
* status，step节点状态"wait"、"process"、"finish"、"error"；优先级高于Steps组件的current属性
* icon，图标节点，可接受字符串或react节点
* title，step节点的标题
* description，step节点的描述

## 使用

    import {Component} from "react";
    import {Steps} from "antd";
    const Step = Steps.Step;
    
    class MySteps extends Component{
      render(){
        return (
          <Steps>
            <Step title="Finished" description="This is a description." />
            <Step title="In Progress" description="This is a description." />
            <Step title="Waiting" description="This is a description." />
          </Steps>
        )
      }
    }
    
    export default MySteps

## 源码

    'use strict';
    
    Object.defineProperty(exports, "__esModule", {
        value: true
    });
    exports["default"] = undefined;
    
    // 校验是否以构造函数调用
    var _classCallCheck2 = require('babel-runtime/helpers/classCallCheck');
    var _classCallCheck3 = _interopRequireDefault(_classCallCheck2);
    
    var _possibleConstructorReturn2 = require('babel-runtime/helpers/possibleConstructorReturn');
    var _possibleConstructorReturn3 = _interopRequireDefault(_possibleConstructorReturn2);
    
    // 继承，私有属性需要执行父类的构造函数
    var _inherits2 = require('babel-runtime/helpers/inherits');
    var _inherits3 = _interopRequireDefault(_inherits2);
    
    var _react = require('react');
    var _react2 = _interopRequireDefault(_react);
    
    var _rcSteps = require('rc-steps');
    var _rcSteps2 = _interopRequireDefault(_rcSteps);
    
    function _interopRequireDefault(obj) { return obj && obj.__esModule ? obj : { "default": obj }; }
    
    // 除设定props默认值和类型外，直接调用"rc-steps"组件作渲染
    var Steps = function (_React$Component) {
        (0, _inherits3["default"])(Steps, _React$Component);
    
        function Steps() {
            (0, _classCallCheck3["default"])(this, Steps);
            return (0, _possibleConstructorReturn3["default"])(this, _React$Component.apply(this, arguments));
        }
    
        Steps.prototype.render = function render() {
            return _react2["default"].createElement(_rcSteps2["default"], this.props);
        };
    
        return Steps;
    }(_react2["default"].Component);
    
    exports["default"] = Steps;
    
    Steps.Step = _rcSteps2["default"].Step;
    Steps.defaultProps = {
        prefixCls: 'ant-steps',
        iconPrefix: 'ant',
        current: 0
    };
    Steps.propTypes = {
        prefixCls: _react.PropTypes.string,
        iconPrefix: _react.PropTypes.string,
        current: _react.PropTypes.number
    };
    module.exports = exports['default'];

