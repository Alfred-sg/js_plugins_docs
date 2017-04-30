# dva 1.2.1

[文档地址](https://github.com/dvajs/dva)

## 概述

dva基于redux、react-redux管理state，基于react-router、histrory配置路由，基于redux-saga调控异步流程。

dva通过构建model配置redux中的reducers，redux-saga中的effects，同时实现不同的命名空间。

## 基本使用

取dva官方示例antd-admin-master

### 设置model

	import { login, userInfo, logout } from '../services/app'
	import { parse } from 'qs'
	
	export default {
	  // 设置命名空间，react组件中派发action为{type:"app/login"}形式
	  namespace: 'app',
	  
	  // 初始化state
	  state: {
	    login: false,
	    user: {
	      name: '吴彦祖',
	    },
	    loginButtonLoading: false,
	    menuPopoverVisible: false,
	    siderFold: localStorage.getItem('antdAdminSiderFold') === 'true',
	    darkTheme: localStorage.getItem('antdAdminDarkTheme') !== 'false',
	    isNavbar: document.body.clientWidth < 769,
	    navOpenKeys: [],
	    permissions: {
	      dashboard: {
	        text: 'Dashboard',
	        route: 'dashboard',
	      },
	      users: {
	        text: 'User Manage',
	        route: 'users',
	      },
	      UIElement: {
	        text: 'UI Element',
	        route: 'UIElement',
	      },
	      UIElementIconfont: {
	        text: 'Iconfont',
	        route: 'UIElement/iconfont',
	        parent: 'UIElement',
	      },
	      chart: {
	        text: 'Rechart',
	        route: 'chart',
	      },
	    },
	    userPermissions: [],
	  },
	  
	  // 应用启动时当即调用函数
	  subscriptions: {
	    setup ({ dispatch }) {
	      dispatch({ type: 'queryUser' })
	      window.onresize = () => {
	        dispatch({ type: 'changeNavbar' })
	      }
	    },
	  },
	  
	  // redux-saga机制设置的effect，用于主控异步流程，react组件中可通过dispatch({type:"app/login"})形式派发
	  effects: {
	    *login ({
	      payload,
	    }, { call, put }) {
	      yield put({ type: 'showLoginButtonLoading' })
	      const { success, userPermissions, username } = yield call(login, parse(payload))
	      if (success) {
	        yield put({
	          type: 'loginSuccess',
	          payload: {
	            userPermissions,
	            user: {
	              name: username,
	            },
	          } })
	      } else {
	        yield put({
	          type: 'loginFail',
	        })
	      }
	    },
	    *queryUser ({
	      payload,
	    }, { call, put }) {
	      const { success, userPermissions, username } = yield call(userInfo, parse(payload))
	      if (success) {
	        yield put({
	          type: 'loginSuccess',
	          payload: {
	            userPermissions,
	            user: {
	              name: username,
	            },
	          },
	        })
	      }
	    },
	    *logout ({
	      payload,
	    }, { call, put }) {
	      const data = yield call(logout, parse(payload))
	      if (data.success) {
	        yield put({
	          type: 'logoutSuccess',
	        })
	      }
	    },
	    *switchSider ({
	      payload,
	    }, { put }) {
	      yield put({
	        type: 'handleSwitchSider',
	      })
	    },
	    *changeTheme ({
	      payload,
	    }, { put }) {
	      yield put({
	        type: 'handleChangeTheme',
	      })
	    },
	    *changeNavbar ({
	      payload,
	    }, { put }) {
	      if (document.body.clientWidth < 769) {
	        yield put({ type: 'showNavbar' })
	      } else {
	        yield put({ type: 'hideNavbar' })
	      }
	    },
	    *switchMenuPopver ({
	      payload,
	    }, { put }) {
	      yield put({
	        type: 'handleSwitchMenuPopver',
	      })
	    },
	  },
	  
	  // redux机制设置的reducers，react组价中可通过dispatch({type:"loginSuccess"})派发action
	  reducers: {
	    loginSuccess (state, action) {
	      return {
	        ...state,
	        ...action.payload,
	        login: true,
	        loginButtonLoading: false,
	      }
	    },
	    logoutSuccess (state) {
	      return {
	        ...state,
	        login: false,
	      }
	    },
	    loginFail (state) {
	      return {
	        ...state,
	        login: false,
	        loginButtonLoading: false,
	      }
	    },
	    showLoginButtonLoading (state) {
	      return {
	        ...state,
	        loginButtonLoading: true,
	      }
	    },
	    handleSwitchSider (state) {
	      localStorage.setItem('antdAdminSiderFold', !state.siderFold)
	      return {
	        ...state,
	        siderFold: !state.siderFold,
	      }
	    },
	    handleChangeTheme (state) {
	      localStorage.setItem('antdAdminDarkTheme', !state.darkTheme)
	      return {
	        ...state,
	        darkTheme: !state.darkTheme,
	      }
	    },
	    showNavbar (state) {
	      return {
	        ...state,
	        isNavbar: true,
	      }
	    },
	    hideNavbar (state) {
	      return {
	        ...state,
	        isNavbar: false,
	      }
	    },
	    handleSwitchMenuPopver (state) {
	      return {
	        ...state,
	        menuPopoverVisible: !state.menuPopoverVisible,
	      }
	    },
	    handleNavOpenKeys (state, action) {
	      return {
	        ...state,
	        ...action.payload,
	      }
	    },
	  },
	}

### 设置路由

	import React, { PropTypes } from 'react'
	import { Router } from 'dva/router'
	// import pathToRegexp from 'path-to-regexp'
	import App from './routes/app'
	
	const cached = {}
	const registerModel = (app, model) => {
	  if (!cached[model.namespace]) {
	    app.model(model)
	    cached[model.namespace] = 1
	  }
	}
	
	const Routers = function ({ history, app }) {
	  const handleChildRoute = ({ location, params, routes }) => {
	    console.log(location, params, routes)
	  }
	
	  const routes = [
	    {
	      path: '/',
	      component: App,
	      getIndexRoute (nextState, cb) {
	        require.ensure([], require => {
	          registerModel(app, require('./models/dashboard'))
	          cb(null, { component: require('./routes/dashboard/') })
	        }, 'dashboard')
	      },
	      childRoutes: [
	        {
	          path: 'dashboard',
	          getComponent (nextState, cb) {
	            require.ensure([], require => {
	              registerModel(app, require('./models/dashboard'))
	              cb(null, require('./routes/dashboard/'))
	            }, 'dashboard')
	          },
	        }, {
	          path: 'users',
	          getComponent (nextState, cb) {
	            require.ensure([], require => {
	              registerModel(app, require('./models/users'))
	              cb(null, require('./routes/users/'))
	            }, 'users')
	          },
	        }, {
	          path: 'request',
	          getComponent (nextState, cb) {
	            require.ensure([], require => {
	              cb(null, require('./routes/request/'))
	            }, 'request')
	          },
	        }, {
	          path: 'UIElement/iconfont',
	          getComponent (nextState, cb) {
	            require.ensure([], require => {
	              cb(null, require('./routes/UIElement/iconfont/'))
	            }, 'UIElement-iconfont')
	          },
	        }, {
	          path: 'UIElement/search',
	          getComponent (nextState, cb) {
	            require.ensure([], require => {
	              cb(null, require('./routes/UIElement/search/'))
	            }, 'UIElement-search')
	          },
	        }, {
	          path: 'UIElement/dropOption',
	          getComponent (nextState, cb) {
	            require.ensure([], require => {
	              cb(null, require('./routes/UIElement/dropOption/'))
	            }, 'UIElement-dropOption')
	          },
	        }, {
	          path: 'UIElement/layer',
	          getComponent (nextState, cb) {
	            require.ensure([], require => {
	              cb(null, require('./routes/UIElement/layer/'))
	            }, 'UIElement-layer')
	          },
	        }, {
	          path: 'UIElement/dataTable',
	          getComponent (nextState, cb) {
	            require.ensure([], require => {
	              cb(null, require('./routes/UIElement/dataTable/'))
	            }, 'UIElement-dataTable')
	          },
	        }, {
	          path: 'UIElement/editor',
	          getComponent (nextState, cb) {
	            require.ensure([], require => {
	              cb(null, require('./routes/UIElement/editor/'))
	            }, 'UIElement-editor')
	          },
	        }, {
	          path: 'chart/lineChart',
	          getComponent (nextState, cb) {
	            require.ensure([], require => {
	              cb(null, require('./routes/chart/lineChart/'))
	            }, 'chart-lineChart')
	          },
	        }, {
	          path: 'chart/barChart',
	          getComponent (nextState, cb) {
	            require.ensure([], require => {
	              cb(null, require('./routes/chart/barChart/'))
	            }, 'chart-barChart')
	          },
	        }, {
	          path: 'chart/areaChart',
	          getComponent (nextState, cb) {
	            require.ensure([], require => {
	              cb(null, require('./routes/chart/areaChart/'))
	            }, 'chart-areaChart')
	          },
	        }, {
	          path: '*',
	          getComponent (nextState, cb) {
	            require.ensure([], require => {
	              cb(null, require('./routes/error/'))
	            }, 'error')
	          },
	        },
	      ],
	    },
	  ]
	
	  routes[0].childRoutes.map(item => {
	    item.onEnter = handleChildRoute
	    return item
	  })
	
	  return <Router history={history} routes={routes} />
	}
	
	Routers.propTypes = {
	  history: PropTypes.object,
	  app: PropTypes.object,
	}
	
	export default Routers


### 启动应用

	import './index.html'
	// 须配置babel，以编译生成器函数
	import 'babel-polyfill'
	import dva from 'dva'
	import createLoading from 'dva-loading'
	import { browserHistory } from 'dva/router'
	
	// 1. Initialize
	const app = dva({
	  // createLoading用于添加额外的reducers，封装effects
	  ...createLoading(),
	  
	  // react-router挂载的histroy对象
	  history: browserHistory,
	  
	  // 报错时打印提示
	  onError (error) {
	    console.error('app onError -- ', error)
	  },
	})
	
	// 2. Model 设置model，约定命名空间，初始化state，设置reducers、effects，以更新state
	app.model(require('./models/app'))
	
	// 3. Router 配置路由
	app.router(require('./router'))
	
	// 4. Start

## 核心源码

### createDva.js挂载plugin、model、router，启动应用

	'use strict';
	
	Object.defineProperty(exports, "__esModule", {
	  value: true
	});
	
	var _regenerator = require('babel-runtime/regenerator');
	var _regenerator2 = _interopRequireDefault(_regenerator);
	
	var _typeof2 = require('babel-runtime/helpers/typeof');
	var _typeof3 = _interopRequireDefault(_typeof2);
	
	var _toConsumableArray2 = require('babel-runtime/helpers/toConsumableArray');
	var _toConsumableArray3 = _interopRequireDefault(_toConsumableArray2);
	
	var _keys = require('babel-runtime/core-js/object/keys');
	var _keys2 = _interopRequireDefault(_keys);
	
	// 浅拷贝
	var _extends2 = require('babel-runtime/helpers/extends');
	var _extends3 = _interopRequireDefault(_extends2);
	
	var _getIterator2 = require('babel-runtime/core-js/get-iterator');
	var _getIterator3 = _interopRequireDefault(_getIterator2);
	
	exports.default = createDva;
	
	var _react = require('react');
	var _react2 = _interopRequireDefault(_react);
	
	var _reactRedux = require('react-redux');
	
	var _redux = require('redux');
	
	var _middleware = require('redux-saga/lib/internal/middleware');
	var _middleware2 = _interopRequireDefault(_middleware);
	
	var _effects = require('redux-saga/effects');
	var sagaEffects = _interopRequireWildcard(_effects);
	
	var _isPlainObject = require('is-plain-object');
	var _isPlainObject2 = _interopRequireDefault(_isPlainObject);
	
	var _invariant = require('invariant');
	var _invariant2 = _interopRequireDefault(_invariant);
	
	var _warning = require('warning');
	var _warning2 = _interopRequireDefault(_warning);
	
	var _flatten = require('flatten');
	var _flatten2 = _interopRequireDefault(_flatten);
	
	var _window = require('global/window');
	var _window2 = _interopRequireDefault(_window);
	
	var _document = require('global/document');
	var _document2 = _interopRequireDefault(_document);
	
	var _sagaHelpers = require('redux-saga/lib/internal/sagaHelpers');
	
	var _lodash = require('lodash.isfunction');
	var _lodash2 = _interopRequireDefault(_lodash);
	
	var _handleActions = require('./handleActions');
	var _handleActions2 = _interopRequireDefault(_handleActions);
	
	var _plugin = require('./plugin');
	var _plugin2 = _interopRequireDefault(_plugin);
	
	function _interopRequireWildcard(obj) { 
	  if (obj && obj.__esModule) { 
	    return obj; 
	  } else { 
	    var newObj = {}; 
	    if (obj != null) { 
	      for (var key in obj) { 
	        if (Object.prototype.hasOwnProperty.call(obj, key)) newObj[key] = obj[key]; 
	      } 
	    } 
	    newObj.default = obj; return newObj; 
	  } 
	}
	
	function _interopRequireDefault(obj) { 
	  return obj && obj.__esModule ? obj : { default: obj }; 
	}
	
	var SEP = '/';
	
	function createDva(createOpts) {
	  var mobile = createOpts.mobile,
	      initialReducer = createOpts.initialReducer,// 默认require('react-router-redux').routerReducer
	      defaultHistory = createOpts.defaultHistory,// 默认'react-router/lib/hashHistory'
	      routerMiddleware = createOpts.routerMiddleware,// 默认require('react-router-redux').routerMiddleware
	      setupHistory = createOpts.setupHistory;
	
	  /**
	   * Create a dva instance.
	   */
	
	  return function dva() {
	    var hooks = arguments.length > 0 && arguments[0] !== undefined ? arguments[0] : {};
	
	    // history and initialState does not pass to plugin
	    var history = hooks.history || defaultHistory;
	    var initialState = hooks.initialState || {};
	    delete hooks.history;
	    delete hooks.initialState;
	
	    var plugin = new _plugin2.default();
	    plugin.use(hooks);
	
	    var app = {
	      // properties
	      _models: [],
	      _router: null,
	      _store: null,
	      _history: null,
	      _plugin: plugin,
	      // methods
	
	      // dva.use(plugin={key:fn})添加钩子函数
	      // 当key为'extraEnhancers'，值须为数组形式，该值将替换plugin.hooks.extraEnhancers，用于添加redux中间件
	      //    当key为其他时，名义上没有限制类型，且使用push方法添加到plugin.hooks[key]中
	      // 当key为'extraReducers'，值须为{actionType:reducer()=>{}}，且actionType不能和任意model对象的命名空间重名
	      //    用于添加额外的reducer
	      // 当key为'onReducer'时，值须为fn，用于装饰redux.combineReducers返回值
	      // 当key为'onAction'时，值须为fn，用于添加redux中间件如redux-logger等
	      // 当key为'onStateChange'时，值须为fn，redux机制的dispatch方法执行修改state时，触发调用fn监听函数
	      // 当key为'onEffect'时，值须为fn(effect,sagaEffects,model,key)，用于装饰model.effects中各生成器函数
	      use: use,
	
	      // dva.model({namespace,state,subscriptions,reducers,effects})，向this._models添加经属性名转换后的model对象
	      // 参数model.reducers={actionType:reducer(state,action)=>{}}
	      //    或者model.reducers=[{actionType:reducer(state,action)=>{}},wrap((state,action)=>{}){}]
	      //    前者单纯设定actionType该调用执行的reducer以修改state
	      //    后者除设定actionType下该调用的reducer以外，还可对handleActions流程控制函数作包装
	      model: model,
	      
	      // dva.router(fn)形式配置路由规则，this._router=router
	      router: router,
	      start: start
	    };
	    return app;
	
	    // //////////////////////////////////
	    // Methods
	
	    // dva.use(plugin={key:fn})添加钩子函数
	    function use(hooks) {
	      plugin.use(hooks);
	    }
	
	    // dva.model({ namespace, state, subscriptions, reducers, effects })向this._models添加经属性名转换后的model对象
	    // model.effects可以是对象或数组，对象其属性值为生成器函数，参数为封装put方法后的sagaEffects对象
	    //    若为数组，第二项为配置项opts，且
	    //        opts.type须是['watcher', 'takeEvery', 'takeLatest', 'throttle']中的一个，约定saga构建方案
	    //        opts.type若为'throttle'，则opts.ms必须存在
	    function model(model) {
	      // checkModel函数校验model模块的namespace、subscriptions、reducers、effects属性
	      //    同时将model.reducers、model.reducers[0]、model.effects对象的属性名转换为namespace+"/"+key形式
	      // 经过属性名转换后的model对象存入this._models中
	      this._models.push(checkModel(model, mobile));
	    }
	
	    // 由bind方法将injectModel赋值给this.model，用于注入特定的m
	    function injectModel(createReducer, onError, unlisteners, m) {
	      m = checkModel(m, mobile);
	      this._models.push(m);
	      var store = this._store;
	
	      // reducers
	      // getReducer函数获取根据action.type调用相应model.reducer[action.type]以修改state的的流程控制函数；或者顺序传递state
	      store.asyncReducers[m.namespace] = getReducer(m.reducers, m.state);
	      store.replaceReducer(createReducer(store.asyncReducers));
	      // effects
	      if (m.effects) {
	        store.runSaga(getSaga(m.effects, m, onError));
	      }
	      // subscriptions
	      if (m.subscriptions) {
	        unlisteners[m.namespace] = runSubscriptions(m.subscriptions, m, this, onError);
	      }
	    }
	
	    // 由bind方法将unmodel赋值给this.unmodel，用于以namespace移除特定的model
	    function unmodel(createReducer, reducers, _unlisteners, namespace) {
	      var store = this._store;
	
	      // Delete reducers
	      delete store.asyncReducers[namespace];
	      delete reducers[namespace];
	      store.replaceReducer(createReducer(store.asyncReducers));
	      store.dispatch({ type: '@@dva/UPDATE' });
	
	      // Cancel effects
	      store.dispatch({ type: namespace + '/@@CANCEL_EFFECTS' });
	
	      // unlisten subscrioptions
	      if (_unlisteners[namespace]) {
	        var _unlisteners$namespac = _unlisteners[namespace],
	            unlisteners = _unlisteners$namespac.unlisteners,
	            noneFunctionSubscriptions = _unlisteners$namespac.noneFunctionSubscriptions;
	
	        (0, _warning2.default)(noneFunctionSubscriptions.length === 0, 'app.unmodel: subscription should return unlistener function, check these subscriptions ' + noneFunctionSubscriptions.join(', '));
	        var _iteratorNormalCompletion = true;
	        var _didIteratorError = false;
	        var _iteratorError = undefined;
	
	        try {
	          for (var _iterator = (0, _getIterator3.default)(unlisteners), _step; !(_iteratorNormalCompletion = (_step = _iterator.next()).done); _iteratorNormalCompletion = true) {
	            var unlistener = _step.value;
	
	            unlistener();
	          }
	        } catch (err) {
	          _didIteratorError = true;
	          _iteratorError = err;
	        } finally {
	          try {
	            if (!_iteratorNormalCompletion && _iterator.return) {
	              _iterator.return();
	            }
	          } finally {
	            if (_didIteratorError) {
	              throw _iteratorError;
	            }
	          }
	        }
	
	        delete _unlisteners[namespace];
	      }
	
	      // delete model from this._models
	      this._models = this._models.filter(function (model) {
	        return model.namespace !== namespace;
	      });
	    }
	
	    // dva.router(fn)形式配置路由规则
	    function router(router) {
	      (0, _invariant2.default)(typeof router === 'function', 'app.router: router should be function');
	      this._router = router;
	    }
	
	    // 启动，转化redux配置，添加router配置，挂载渲染组件或返回组件
	    function start(container) {
	      // 通过container获取容器节点，并校验该容器节点为html元素
	      if (typeof container === 'string') {
	        container = _document2.default.querySelector(container);
	        (0, _invariant2.default)(container, 'app.start: could not query selector: ' + container);
	      }
	      (0, _invariant2.default)(!container || isHTMLElement(container), 
	        'app.start: container should be HTMLElement');
	
	      // 校验dva.router(fn)方法得到执行，即this._router有值
	      (0, _invariant2.default)(this._router, 'app.start: router should be defined');
	
	      // plugin.apply('onError',defaultHandler)注册默认"onError"事件处理函数defaultHandler，即抛出错误
	      // 若客户使用dva.use({onError:fn})注册"onError"事件处理钩子时，onError函数执行时将调用fn(err,dispatch)函数，同时不调用defaultHandler
	      var onError = plugin.apply('onError', function (err) {
	        throw new Error(err.stack || err);
	      });
	      var onErrorWrapper = function onErrorWrapper(err) {
	        if (err) {
	          if (typeof err === 'string') err = new Error(err);
	          onError(err, app._store.dispatch);
	        }
	      };
	
	      // internal model for destroy
	      // dva.model({ namespace, state, subscriptions, reducers, effects })向this._models添加经属性名转换后的model对象
	      model.call(this, {
	        namespace: '@@dva',
	        state: 0,
	        reducers: {
	          UPDATE: function UPDATE(state) {
	            return state + 1;
	          }
	        }
	      });
	
	      // get reducers and sagas from model
	      var sagas = [];
	      var reducers = (0, _extends3.default)({}, initialReducer);// 存储各model的流程控制函数
	      var _iteratorNormalCompletion2 = true;
	      var _didIteratorError2 = false;
	      var _iteratorError2 = undefined;
	
	      // 以迭代器形式将this._models中各model.reducers转化为流程控制函数，添加到reducers[model.namespace]中
	      // 流程控制函数中，当传参action.type与model.reducers的属性名相符，修改state；不符，顺序传递state
	      try {
	        for (var _iterator2 = (0, _getIterator3.default)(this._models), _step2; !(_iteratorNormalCompletion2 = (_step2 = _iterator2.next()).done); _iteratorNormalCompletion2 = true) {
	          var m = _step2.value;
	
	          // getReducer函数获取根据action.type调用相应model.reducer[action.type]以修改state的的流程控制函数
	          // 或者顺序传递state
	          reducers[m.namespace] = getReducer(m.reducers, m.state);
	
	          // getSaga函数构建生成器函数，迭代model.effects各生成器函数，设置action并作相应处理
	          if (m.effects) sagas.push(getSaga(m.effects, m, onErrorWrapper));
	        }
	      } catch (err) {
	        _didIteratorError2 = true;
	        _iteratorError2 = err;
	      } finally {
	        try {
	          if (!_iteratorNormalCompletion2 && _iterator2.return) {
	            _iterator2.return();
	          }
	        } finally {
	          if (_didIteratorError2) {
	            throw _iteratorError2;
	          }
	        }
	      }
	
	      // dva.use({'extraReducers':{actionType:reducer()=>{}}})以数组形式将fn存储在plugin.hooks.extraReducers中
	      // 即plugin.hooks.extraReducers数组元素各对象的属性名不能与任意model对象的命名空间重名
	      var extraReducers = plugin.get('extraReducers');
	      (0, _invariant2.default)((0, _keys2.default)(extraReducers).every(function (key) {
	        return !(key in reducers);
	      }), 'app.start: extraReducers is conflict with other reducers');
	
	      // dva.use({'extraReducers':[]})替换plugin.hooks.extraEnhancers形式添加'extraEnhancers'钩子
	      var extraEnhancers = plugin.get('extraEnhancers');
	      (0, _invariant2.default)(Array.isArray(extraEnhancers), 
	        'app.start: extraEnhancers should be array');
	
	      // 创建redux的中间件middlewares，使用redux.applyMiddleware方法完成挂载
	      var extraMiddlewares = plugin.get('onAction');
	      var reducerEnhancer = plugin.get('onReducer');
	      var sagaMiddleware = (0, _middleware2.default)();// redux中间件机制为着装饰dispatch方法
	      var middlewares = [sagaMiddleware].concat((0, _toConsumableArray3.default)((0, _flatten2.default)(extraMiddlewares)));
	      if (routerMiddleware) {
	        middlewares = [routerMiddleware(history)].concat((0, _toConsumableArray3.default)(middlewares));
	      }
	      var devtools = function devtools() {
	        return function (noop) {
	          return noop;
	        };
	      };
	      if (process.env.NODE_ENV !== 'production' && _window2.default.__REDUX_DEVTOOLS_EXTENSION__) {
	        devtools = _window2.default.__REDUX_DEVTOOLS_EXTENSION__;
	      }
	      var enhancers = [_redux.applyMiddleware.apply(undefined, (0, _toConsumableArray3.default)(middlewares)), devtools()].concat((0, _toConsumableArray3.default)(extraEnhancers));
	      
	      // 创建redux的store
	      var store = this._store = (0, _redux.createStore)(createReducer(), initialState, _redux.compose.apply(undefined, (0, _toConsumableArray3.default)(enhancers)));
	
	      // 通过plugin.hooks.onReducer数组中函数装饰redux.combineReducers返回值
	      function createReducer(asyncReducers) {
	        return reducerEnhancer((0, _redux.combineReducers)((0, _extends3.default)({}, reducers, extraReducers, asyncReducers)));
	      }
	
	      // extend store
	      store.runSaga = sagaMiddleware.run;
	      store.asyncReducers = {};
	
	      // dva.use({onStateChange:fn()=>{}})绑定dispatch执行修改state时将触发的fn监听函数
	      var listeners = plugin.get('onStateChange');
	      var _iteratorNormalCompletion3 = true;
	      var _didIteratorError3 = false;
	      var _iteratorError3 = undefined;
	
	      // store.subscribe(fn)方法当dispatch方法执行修改state时调用fn
	      try {
	        for (var _iterator3 = (0, _getIterator3.default)(listeners), _step3; !(_iteratorNormalCompletion3 = (_step3 = _iterator3.next()).done); _iteratorNormalCompletion3 = true) {
	          var listener = _step3.value;
	
	          store.subscribe(listener);
	        }
	      } catch (err) {
	        _didIteratorError3 = true;
	        _iteratorError3 = err;
	      } finally {
	        try {
	          if (!_iteratorNormalCompletion3 && _iterator3.return) {
	            _iterator3.return();
	          }
	        } finally {
	          if (_didIteratorError3) {
	            throw _iteratorError3;
	          }
	        }
	      }
	
	      // require('redux-saga/lib/internal/middleware').middleware添加redux-saga中间件
	      // sagaMiddleware.run(sage)设定redux-saga中间件的运行逻辑
	      // redux-saga机制？？？
	      sagas.forEach(sagaMiddleware.run);
	
	      // 设定history
	      if (setupHistory) setupHistory.call(this, history);
	
	      // 组件渲染完成当即执行model.subscription中各方法，派发action，返回解绑函数集合
	      var unlisteners = {};
	      var _iteratorNormalCompletion4 = true;
	      var _didIteratorError4 = false;
	      var _iteratorError4 = undefined;
	
	      try {
	        for (var _iterator4 = (0, _getIterator3.default)(this._models), _step4; !(_iteratorNormalCompletion4 = (_step4 = _iterator4.next()).done); _iteratorNormalCompletion4 = true) {
	          var _model = _step4.value;
	
	          if (_model.subscriptions) {
	            unlisteners[_model.namespace] = runSubscriptions(_model.subscriptions, _model, this, onErrorWrapper);
	          }
	        }
	      } catch (err) {
	        _didIteratorError4 = true;
	        _iteratorError4 = err;
	      } finally {
	        try {
	          if (!_iteratorNormalCompletion4 && _iterator4.return) {
	            _iterator4.return();
	          }
	        } finally {
	          if (_didIteratorError4) {
	            throw _iteratorError4;
	          }
	        }
	      }
	
	      this.model = injectModel.bind(this, createReducer, onErrorWrapper, unlisteners);
	
	      this.unmodel = unmodel.bind(this, createReducer, reducers, unlisteners);
	
	      // 若容器节点存在，挂载组件；不存在，返回react组件
	      if (container) {
	        render(container, store, this, this._router);
	        plugin.apply('onHmr')(render.bind(this, container, store, this));
	      } else {
	        return getProvider(store, this, this._router);
	      }
	    }
	
	    // //////////////////////////////////
	    // Helpers
	
	    // 返回react组件
	    function getProvider(store, app, router) {
	      return function (extraProps) {
	        return _react2.default.createElement(
	          _reactRedux.Provider,
	          { store: store },
	          router((0, _extends3.default)({ app: app, history: app._history }, extraProps))
	        );
	      };
	    }
	
	    // 挂载组件
	    function render(container, store, app, router) {
	      var ReactDOM = require('react-dom');
	      ReactDOM.render(_react2.default.createElement(getProvider(store, app, router)), container);
	    }
	
	    // 校验model模块的namespace、subscriptions、reducers、effects属性
	    // 同时将model.reducers、model.reducers[0]、model.effects对象的属性名转换为namespace+"/"+key形式
	    function checkModel(m, mobile) {
	      var model = (0, _extends3.default)({}, m);
	      var namespace = model.namespace,
	          reducers = model.reducers,
	          effects = model.effects;
	
	      // 校验model.namespace属性不能为空，不能重复，且mobile为否值时不能赋值为'routing'
	      (0, _invariant2.default)(namespace, 'app.model: namespace should be defined');
	      (0, _invariant2.default)(!app._models.some(function (model) {
	        return model.namespace === namespace;
	      }), 'app.model: namespace should be unique');
	      (0, _invariant2.default)(mobile || namespace !== 'routing', 
	        'app.model: namespace should not be routing, it\'s used by react-redux-router');
	      
	      // 校验model.subscriptions属性为普通对象
	      (0, _invariant2.default)(!model.subscriptions || (0, _isPlainObject2.default)(model.subscriptions), 'app.model: subscriptions should be Object');
	      
	      // 校验model.reducers属性须为对象或数组
	      (0, _invariant2.default)(!reducers || (0, _isPlainObject2.default)(reducers) || 
	        Array.isArray(reducers), 'app.model: reducers should be Object or array');
	      // 校验当model.reducers为数组时，其首项须为对象，第二项须为函数
	      (0, _invariant2.default)(!Array.isArray(reducers) || (0, _isPlainObject2.default)(reducers[0]) 
	        && typeof reducers[1] === 'function', 
	        'app.model: reducers with array should be app.model({ reducers: [object, function] })');
	
	      // 校验model.effects须为对象
	      (0, _invariant2.default)(!effects || (0, _isPlainObject2.default)(effects), 
	        'app.model: effects should be Object');
	
	      function applyNamespace(type) {
	        // 将model.reducers、model.reducers[0]、model.effects对象的属性名转换为namespace+"/"+key形式
	        function getNamespacedReducers(reducers) {
	          return (0, _keys2.default)(reducers).reduce(function (memo, key) {
	            // 校验model.reducers、model.reducers[0]、model.effects对象的属性不能以namespace+"/"起始
	            (0, _warning2.default)(key.indexOf('' + namespace + SEP) !== 0, 
	              'app.model: ' + type.slice(0, -1) + ' ' + key + 
	                ' should not be prefixed with namespace ' + namespace);
	            memo['' + namespace + SEP + key] = reducers[key];
	            return memo;
	          }, {});
	        }
	
	        if (model[type]) {
	          if (type === 'reducers' && Array.isArray(model[type])) {
	            model[type][0] = getNamespacedReducers(model[type][0]);
	          } else {
	            model[type] = getNamespacedReducers(model[type]);
	          }
	        }
	      }
	
	      // 将model.reducers或model.reducers[0]对象的属性名转换为namespace+"/"+key形式
	      applyNamespace('reducers');
	      // 将model.effects对象的属性名转换为namespace+"/"+key形式
	      applyNamespace('effects');
	
	      return model;
	    }
	
	    // 是否html节点元素
	    function isHTMLElement(node) {
	      return (typeof node === 'undefined' ? 'undefined' : 
	        (0, _typeof3.default)(node)) === 'object' && node !== null && node.nodeType && node.nodeName;
	    }
	
	    // 获取根据action.type调用相应model.reducer[action.type]以修改state的的流程控制函数；或者顺序传递state
	    function getReducer(reducers, state) {
	      // model.reducers为数组时，调用model.reducers[1]包装handleActions流程控制函数的返回值
	      // handleActions流程控制函数用于设定特定action.type下该调用的reducer函数以修改state
	      //    其返回值为函数(state,action)=>{}，根据action.type调用model.reducers中相应的reducer；或顺序传递state
	      if (Array.isArray(reducers)) {
	        return reducers[1]((0, _handleActions2.default)(reducers[0], state));
	      } else {
	        return (0, _handleActions2.default)(reducers || {}, state);
	      }
	    }
	
	    // regenerator.mark(fn)将fn.prototype赋值为迭代器的原型对象并附以next、return、throw方法后返回fn
	    //    若将封装后的fn作为参数outerFn传入regenerator.wrap方法中，wrap内获取fn.prototype迭代器原型，为其添加_invoke方法后返回
	    // regenerator.wrap(innerFn,outerFn,self,tryLocsList)方法获取outerFn.prototype迭代器原型或默认迭代器原型
	    //    为其添加_invoke方法后返回，从而其返回值next方法执行时将调用innerFn特定条件分支，throw方法报错，return执行完成
	    // regenerator.keys(object)遍历取出object对象的属性名集合，调用返回值函数，类迭代形式取出反序排列的该属性名集合
	    // context.delegateYield(iterator,resultName,nextLoc)调用时将使iterator替代innerFn输出内容，通过nextLoc跳回到innerFn取值
	
	    // 以生成器函数逻辑处理model.effects各属性，next方法逐层调用时将获取{context:null,fn:watcher,args:undefined}
	    // watcher为getWatcher函数包装model.effects[key]后的生成器函数，用于设置action，并取出fn=sagaWithCatch
	    // sagaWithCatch函数的逻辑中，报错时将跳转onError函数内
	    function getSaga(effects, model, onError) {
	      return _regenerator2.default.mark(function _callee3() {
	        var _this = this;
	
	        var key;
	        return _regenerator2.default.wrap(function _callee3$(_context3) {
	          while (1) {
	            switch (_context3.prev = _context3.next) {
	              case 0:
	                _context3.t0 = _regenerator2.default.keys(effects);
	
	              case 1:
	                if ((_context3.t1 = _context3.t0()).done) {
	                  _context3.next = 7;
	                  break;
	                }
	
	                key = _context3.t1.value;
	
	                if (!Object.prototype.hasOwnProperty.call(effects, key)) {
	                  _context3.next = 5;
	                  break;
	                }
	
	                // context.delegateYield被调用时将代替_callee3$输出内容，即先取_callee2生成器输出内容
	                //    _callee2生成器中，通过对_context.next赋值，跳出_callee2生成器，并回到_callee3$取值
	                return _context3.delegateYield(_regenerator2.default.mark(function _callee2() {
	                  var watcher, task;
	                  return _regenerator2.default.wrap(function _callee2$(_context2) {
	                    while (1) {
	                      switch (_context2.prev = _context2.next) {
	                        case 0:
	                          // watcher赋值为getWatcher创建的生成器函数
	                          watcher = getWatcher(key, effects[key], model, onError);
	                          _context2.next = 3;
	                          // sagaEffects.fork返回{context:null,fn:watcher,args:undefined}
	                          return sagaEffects.fork(watcher);
	
	                        case 3:
	                          task = _context2.sent;
	                          _context2.next = 6;
	                          return sagaEffects.fork(_regenerator2.default.mark(function _callee() {
	                            return _regenerator2.default.wrap(function _callee$(_context) {
	                              while (1) {
	                                switch (_context.prev = _context.next) {
	                                  case 0:
	                                    _context.next = 2;
	                                    // sagaEffects.take返回{[IO]:true,TAKE:model.namespace+'/@@CANCEL_EFFECTS'}
	                                    return sagaEffects.take(model.namespace + '/@@CANCEL_EFFECTS');
	
	                                  case 2:
	                                    _context.next = 4;
	                                    // sagaEffects.cancel返回{[IO]:true,CANCEL:task}
	                                    // task即_callee2$条件分支case 0下返回值{context:null,fn:watcher,args:undefined}
	                                    return sagaEffects.cancel(task);
	
	                                  case 4:
	                                  case 'end':
	                                    return _context.stop();
	                                }
	                              }
	                            }, _callee, this);
	                          }));
	
	                        case 6:
	                        case 'end':
	                          return _context2.stop();
	                      }
	                    }
	                  }, _callee2, _this);
	                })(), 't2', 5);
	
	              // next方法再次调用时，model.effects其中一个属性已经处理完成，跳回到条件分支1，处理下一个model.effects属性
	              case 5:
	                _context3.next = 1;
	                break;
	
	              case 7:
	              case 'end':
	                return _context3.stop();
	            }
	          }
	        }, _callee3, this);
	      });
	    }
	
	    // 构建生成器函数
	    // 当type="watcher"，返回sagaWithCatch生成器函数，其返回值的next方法调用时将执行effect生成器函数
	    // 当type="takeEvery"，返回值的next方法二次调用时将设定action，并输出{context,fn:sagaWithCatch,args:action}
	    function getWatcher(key, _effect, model, onError) {
	      var _marked = [sagaWithCatch].map(_regenerator2.default.mark);
	
	      var effect = _effect;
	      var type = 'takeEvery';
	      var ms = void 0;
	
	      // model.effect为数组时，第二项为配置项opts
	      //    opts.type须是['watcher', 'takeEvery', 'takeLatest', 'throttle']中的一个
	      //    opts.type若为'throttle'，则opts.ms必须存在
	      if (Array.isArray(_effect)) {
	        effect = _effect[0];
	        var opts = _effect[1];
	        if (opts && opts.type) {
	          type = opts.type;
	          if (type === 'throttle') {
	            (0, _invariant2.default)(opts.ms, 'app.start: opts.ms should be defined if type is throttle');
	            ms = opts.ms;
	          }
	        }
	        (0, _invariant2.default)(['watcher', 'takeEvery', 'takeLatest', 'throttle'].indexOf(type) > -1, 
	          'app.start: effect type should be takeEvery, takeLatest, throttle or watcher');
	      }
	
	      // 跟编译后的生成器函数相似，调用sagaWithCatch获取regenerator.wrap装饰后的generator函数
	      // 调用返回值generator函数的next方法，执行effect生成器函数；若报错，则使用onError函数捕获错误
	      // generator函数的next方法的返回值逐步调用next方法，即执行effect生成器函数内yield语句
	      function sagaWithCatch() {
	        var _len,
	            args,
	            _key,
	            _args4 = arguments;
	
	        // effect生成器函数本身经过babel编译
	        // 再使用regenerator.wrap方法包裹，用于捕获effect生成器函数调用时的报错
	        return _regenerator2.default.wrap(function sagaWithCatch$(_context4) {
	          while (1) {
	            switch (_context4.prev = _context4.next) {
	              // sagaWithCatch()返回值首次执行next方法时，调用innerFn即sagaWithCatch的case 0条件分支语句
	              case 0:
	                _context4.prev = 0;
	
	                for (_len = _args4.length, args = Array(_len), _key = 0; _key < _len; _key++) {
	                  args[_key] = _args4[_key];
	                }
	
	                _context4.next = 4;
	
	                // 用户配置的model.effects生成器函数实际获得的参数为args和封装put方法后的sagaEffects对象
	                // createEffects(model)函数获取put方法经封装后的sagaEffects对象
	                // 其put方法输出extend(action,{type:prefixType(type,model)})，即action.type添加model.namespace+"/"前缀
	                return effect.apply(undefined, 
	                    (0, _toConsumableArray3.default)(args.concat(createEffects(model)))
	                  );
	
	              // sagaWithCatch()返回值二次执行next方法时，调用innerFn即sagaWithCatch$的case 4条件分支语句
	              case 4:
	                _context4.next = 9;
	                break;
	
	              // 由regenerator.wrap方法的第四个参数设定，报错时调用innerFn即sagaWithCatch$的case 6条件分支语句
	              case 6:
	                _context4.prev = 6;
	                // context['catch'](0)，按regenrator-runtime机制，为innerFn即sagaWithCatch$执行case 0时捕获的错误
	                _context4.t0 = _context4['catch'](0);
	
	                onError(_context4.t0);
	
	              // sagaWithCatch()返回值三次执行next方法时，调用innerFn即sagaWithCatch$的case 9条件分支语句
	              // 即sagaWithCatch类生成器函数执行完毕
	              case 9:
	              case 'end':
	                return _context4.stop();
	            }
	          }
	        }, _marked[0], this, [[0, 6]]);// [[0, 6]]决定innerFn即sagaWithCatch$的条件语句执行机制
	      }
	
	      // 获取dva.use("onEffect",fn)注册的函数fn(effect,sagaEffects,model,key)装饰model.effects中各生成器函数
	      var onEffect = plugin.get('onEffect');
	      var sagaWithOnEffect = applyOnEffect(onEffect, sagaWithCatch, model, key);
	
	      switch (type) {
	        case 'watcher':
	          // 返回sagaWithCatch，调用sagaWithCatch函数创建生成器，next方法执行effect生成器函数
	          // sagaWithCatch作为redux-saga的主任务流程，整个生命周期内存活
	          // 同一action在派发过程中延迟动作，在延迟执行完成前，再次派发该action无效
	          return sagaWithCatch;
	        case 'takeLatest':
	          // sagaWithCatch作为redux-saga的子任务流程，特定action派发时创建
	          // 同一action在派发过程中延迟动作，在延迟执行完成前，再次派发该action将创建并替换新的子任务流程
	          return _regenerator2.default.mark(function _callee4() {
	            return _regenerator2.default.wrap(function _callee4$(_context5) {
	              while (1) {
	                switch (_context5.prev = _context5.next) {
	                  case 0:
	                    _context5.next = 2;
	                    return (0, _sagaHelpers.takeLatestHelper)(key, sagaWithOnEffect);
	
	                  case 2:
	                  case 'end':
	                    return _context5.stop();
	                }
	              }
	            }, _callee4, this);
	          });
	        case 'throttle':
	          // sagaWithCatch作为redux-saga的子任务流程，特定action派发时创建
	          // 同一action在派发过程中延迟动作，延迟ms时间后再次派发该action有效
	          return _regenerator2.default.mark(function _callee5() {
	            return _regenerator2.default.wrap(function _callee5$(_context6) {
	              while (1) {
	                switch (_context6.prev = _context6.next) {
	                  case 0:
	                    _context6.next = 2;
	                    return (0, _sagaHelpers.throttleHelper)(ms, key, sagaWithOnEffect);
	
	                  case 2:
	                  case 'end':
	                    return _context6.stop();
	                }
	              }
	            }, _callee5, this);
	          });
	        default:
	          // sagaWithCatch作为redux-saga的子任务流程，特定action派发时创建
	          // 同一action在派发过程中延迟动作，再次派发该action有效顺序创建另一延迟动作
	
	          // 返回值为generator生成器函数，执行后获取含有next方法的生成器函数相关迭代器
	          // next方法首次执行返回sagaHelpers.takeEveryHelper方法封装的迭代器函数
	          // next方法再次执行意味generator生成器函数执行完毕
	          return _regenerator2.default.mark(function _callee6() {
	            return _regenerator2.default.wrap(function _callee6$(_context7) {
	              while (1) {
	                switch (_context7.prev = _context7.next) {
	                  // _callee6()返回值首次执行next方法时，调用innerFn即_callee6$的case 0条件分支语句
	                  case 0:
	                    _context7.next = 2;
	                    // sagaHelpers.takeEveryHelper方法执行返回迭代器
	                    // 返回值迭代器的next方法首次执行，设定将根据二次调用next方法的参数时更改action值
	                    // next方法再次执行，更改action，并返回{context:null,fn:sagaWithOnEffect,args:action}
	                    return (0, _sagaHelpers.takeEveryHelper)(key, sagaWithOnEffect);
	
	                  // _callee6()返回值二次执行next方法时，调用innerFn即_callee6$的case 2条件分支语句
	                  // 即_callee6类生成器函数执行完毕
	                  case 2:
	                  case 'end':
	                    return _context7.stop();
	                }
	              }
	            }, _callee6, this);
	          });
	      }
	    }
	
	    // 组件渲染完成当即执行model.subscription中各方法，派发action，返回解绑函数集合
	    function runSubscriptions(subs, model, app, onError) {
	      var unlisteners = [];
	      var noneFunctionSubscriptions = [];
	      for (var key in subs) {
	        if (Object.prototype.hasOwnProperty.call(subs, key)) {
	          var sub = subs[key];
	          // 校验model.subscription对象的属性值为函数
	          (0, _invariant2.default)(typeof sub === 'function', 
	            'app.start: subscription should be function');
	          var unlistener = sub({
	            dispatch: createDispatch(app._store.dispatch, model),
	            history: app._history
	          }, onError);
	          if ((0, _lodash2.default)(unlistener)) {
	            unlisteners.push(unlistener);
	          } else {
	            noneFunctionSubscriptions.push(key);
	          }
	        }
	      }
	      return { unlisteners: unlisteners, noneFunctionSubscriptions: noneFunctionSubscriptions };
	    }
	
	    // model.reducers或model.effects存在时，type拼接上model.namespace+"/"前缀后返回
	    function prefixType(type, model) {
	      var prefixedType = '' + model.namespace + SEP + type;
	      if (model.reducers && model.reducers[prefixedType] || model.effects && model.effects[prefixedType]) {
	        return prefixedType;
	      }
	      return type;
	    }
	
	    // 获取put方法经封装后的sagaEffects对象，put方法输出extend(action,{type:prefixType(type,model)})
	    // 即action.type经model.namespace+"/"前缀后
	    function createEffects(model) {
	      function put(action) {
	        var type = action.type;
	
	        // 校验action.type属性必须为真值，且不能以model.namespace+"/"起始
	        (0, _invariant2.default)(type, 'dispatch: action should be a plain Object with type');
	        (0, _warning2.default)(type.indexOf('' + model.namespace + SEP) !== 0, 
	          'effects.put: ' + type + ' should not be prefixed with namespace ' + model.namespace);
	
	        return sagaEffects.put((0, _extends3.default)({}, action, { type: prefixType(type, model) }));
	      }
	      return (0, _extends3.default)({}, sagaEffects, { put: put });
	    }
	
	    // 调用redux的dispatch方法派发action
	    function createDispatch(dispatch, model) {
	      return function (action) {
	        var type = action.type;
	
	        // 校验action.type属性必须为真值，且不能以model.namespace+"/"起始
	        (0, _invariant2.default)(type, 'dispatch: action should be a plain Object with type');
	        (0, _warning2.default)(type.indexOf('' + model.namespace + SEP) !== 0, 
	          'dispatch: ' + type + ' should not be prefixed with namespace ' + model.namespace);
	        return dispatch((0, _extends3.default)({}, action, { type: prefixType(type, model) }));
	      };
	    }
	
	    // 装饰model.effects中各生成器函数
	    // dva.use("onEffect",fn)方法注册的函数fn(effect,sagaEffects,model,key)用于装饰model.effects中各生成器函数
	    function applyOnEffect(fns, effect, model, key) {
	      var _iteratorNormalCompletion5 = true;
	      var _didIteratorError5 = false;
	      var _iteratorError5 = undefined;
	
	      try {
	        for (var _iterator5 = (0, _getIterator3.default)(fns), _step5; !(_iteratorNormalCompletion5 = (_step5 = _iterator5.next()).done); _iteratorNormalCompletion5 = true) {
	          var fn = _step5.value;
	
	          effect = fn(effect, sagaEffects, model, key);
	        }
	      } catch (err) {
	        _didIteratorError5 = true;
	        _iteratorError5 = err;
	      } finally {
	        try {
	          if (!_iteratorNormalCompletion5 && _iterator5.return) {
	            _iterator5.return();
	          }
	        } finally {
	          if (_didIteratorError5) {
	            throw _iteratorError5;
	          }
	        }
	      }
	
	      return effect;
	    }
	  };
	}
	module.exports = exports['default'];
	
### plugin.js插件机制

	'use strict';
	
	Object.defineProperty(exports, "__esModule", {
	  value: true
	});
	
	var _extends2 = require('babel-runtime/helpers/extends');
	var _extends3 = _interopRequireDefault(_extends2);
	
	var _getIterator2 = require('babel-runtime/core-js/get-iterator');
	var _getIterator3 = _interopRequireDefault(_getIterator2);
	
	var _classCallCheck2 = require('babel-runtime/helpers/classCallCheck');
	var _classCallCheck3 = _interopRequireDefault(_classCallCheck2);
	
	var _createClass2 = require('babel-runtime/helpers/createClass');
	var _createClass3 = _interopRequireDefault(_createClass2);
	
	var _isPlainObject = require('is-plain-object');
	var _isPlainObject2 = _interopRequireDefault(_isPlainObject);
	
	var _invariant = require('invariant');
	var _invariant2 = _interopRequireDefault(_invariant);
	
	function _interopRequireDefault(obj) { return obj && obj.__esModule ? obj : { default: obj }; }
	
	var Plugin = function () {
	  function Plugin() {
	    (0, _classCallCheck3.default)(this, Plugin);
	
	    this.hooks = {
	      onError: [],
	      onStateChange: [],
	      onAction: [],
	      onHmr: [],
	      onReducer: [],
	      onEffect: [],
	      extraReducers: [],
	      extraEnhancers: []
	    };
	  }
	
	  // plugin.use(plugin={key:fn})添加钩子函数，存入this.hooks中
	  // 当key为'extraEnhancers'，钩子为单函数形式；当key为其他时，钩子为数组形式
	  (0, _createClass3.default)(Plugin, [{
	    key: 'use',
	    value: function use(plugin) {
	      // 校验plugin位普通对象
	      (0, _invariant2.default)((0, _isPlainObject2.default)(plugin), 
	        'plugin.use: plugin should be plain object');
	      var hooks = this.hooks;
	      for (var key in plugin) {
	        if (Object.prototype.hasOwnProperty.call(plugin, key)) {
	          (0, _invariant2.default)(hooks[key], 'plugin.use: unknown plugin property: ' + key);
	          if (key === 'extraEnhancers') {
	            hooks[key] = plugin[key];
	          } else {
	            hooks[key].push(plugin[key]);
	          }
	        }
	      }
	    }
	
	  // plugin.apple("onError | onHmr",defaultHandler)以迭代器调用数组形式的"onError | onHmr"钩子函数集合
	  // 若不存在this.hooks["onError | onHmr"]，调用defaultHandler函数
	  }, {
	    key: 'apply',
	    value: function apply(key, defaultHandler) {
	      var hooks = this.hooks;
	      var validApplyHooks = ['onError', 'onHmr'];
	      (0, _invariant2.default)(validApplyHooks.indexOf(key) > -1, 
	        'plugin.apply: hook ' + key + ' cannot be applied');
	      var fns = hooks[key];
	
	      return function () {
	        if (fns.length) {
	          var _iteratorNormalCompletion = true;
	          var _didIteratorError = false;
	          var _iteratorError = undefined;
	
	          try {
	            for (var _iterator = (0, _getIterator3.default)(fns), _step; !(_iteratorNormalCompletion = (_step = _iterator.next()).done); _iteratorNormalCompletion = true) {
	              var fn = _step.value;
	
	              fn.apply(undefined, arguments);
	            }
	          } catch (err) {
	            _didIteratorError = true;
	            _iteratorError = err;
	          } finally {
	            try {
	              if (!_iteratorNormalCompletion && _iterator.return) {
	                _iterator.return();
	              }
	            } finally {
	              if (_didIteratorError) {
	                throw _iteratorError;
	              }
	            }
	          }
	        } else if (defaultHandler) {
	          defaultHandler.apply(undefined, arguments);
	        }
	      };
	    }
	
	  // 获取添加的钩子，通常返回plugin.hooks[key]
	  // 当key为'extraReducers'时，plugin.hooks.extraReducers=[{actionType:()=>{}}]从数组转化为对象后输出
	  // 当key为'onReducer'时，plugin.hooks.onReducer=[fn]从数组转化为逐次调用fn的函数，用于装饰redux.combineReducers返回值
	  }, {
	    key: 'get',
	    value: function get(key) {
	      var hooks = this.hooks;
	      (0, _invariant2.default)(key in hooks, 'plugin.get: hook ' + key + ' cannot be got');
	      if (key === 'extraReducers') {
	        var ret = {};
	        var _iteratorNormalCompletion2 = true;
	        var _didIteratorError2 = false;
	        var _iteratorError2 = undefined;
	
	        try {
	          for (var _iterator2 = (0, _getIterator3.default)(hooks[key]), _step2; !(_iteratorNormalCompletion2 = (_step2 = _iterator2.next()).done); _iteratorNormalCompletion2 = true) {
	            var reducerObj = _step2.value;
	
	            ret = (0, _extends3.default)({}, ret, reducerObj);
	          }
	        } catch (err) {
	          _didIteratorError2 = true;
	          _iteratorError2 = err;
	        } finally {
	          try {
	            if (!_iteratorNormalCompletion2 && _iterator2.return) {
	              _iterator2.return();
	            }
	          } finally {
	            if (_didIteratorError2) {
	              throw _iteratorError2;
	            }
	          }
	        }
	
	        return ret;
	      } else if (key === 'onReducer') {
	        return function (reducer) {
	          var _iteratorNormalCompletion3 = true;
	          var _didIteratorError3 = false;
	          var _iteratorError3 = undefined;
	
	          try {
	            for (var _iterator3 = (0, _getIterator3.default)(hooks[key]), _step3; !(_iteratorNormalCompletion3 = (_step3 = _iterator3.next()).done); _iteratorNormalCompletion3 = true) {
	              var reducerEnhancer = _step3.value;
	
	              reducer = reducerEnhancer(reducer);
	            }
	          } catch (err) {
	            _didIteratorError3 = true;
	            _iteratorError3 = err;
	          } finally {
	            try {
	              if (!_iteratorNormalCompletion3 && _iterator3.return) {
	                _iterator3.return();
	              }
	            } finally {
	              if (_didIteratorError3) {
	                throw _iteratorError3;
	              }
	            }
	          }
	
	          return reducer;
	        };
	      } else {
	        return hooks[key];
	      }
	    }
	  }]);
	  return Plugin;
	}();
	
	exports.default = Plugin;
	module.exports = exports['default'];
	
### handlerActions.js管理reducer执行流程

	"use strict";
	
	Object.defineProperty(exports, "__esModule", {
	  value: true
	});
	
	var _toConsumableArray2 = require("babel-runtime/helpers/toConsumableArray");
	var _toConsumableArray3 = _interopRequireDefault(_toConsumableArray2);
	
	var _keys = require("babel-runtime/core-js/object/keys");
	var _keys2 = _interopRequireDefault(_keys);
	
	function _interopRequireDefault(obj) { return obj && obj.__esModule ? obj : { default: obj }; }
	
	function identify(value) {
	  return value;
	}
	
	function handleAction(actionType) {
	  var reducer = arguments.length > 1 && arguments[1] !== undefined ? arguments[1] : identify;
	
	  return function (state, action) {
	    var type = action.type;
	
	    if (type && actionType !== type) {
	      return state;
	    }
	    return reducer(state, action);
	  };
	}
	
	function reduceReducers() {
	  for (var _len = arguments.length, reducers = Array(_len), _key = 0; _key < _len; _key++) {
	    reducers[_key] = arguments[_key];
	  }
	
	  return function (previous, current) {
	    return reducers.reduce(function (p, r) {
	      return r(p, current);
	    }, previous);
	  };
	}
	
	// 参数handlers={actionType:reducer(state,action)=>{}}
	// 流程控制，按返回函数的传参action.type执行相应的reducer，修改state
	function handleActions(handlers, defaultState) {
	  // 获取[(state,action)=>{}]函数集合
	  // 当action.type与actionType相符时，执行相应的reducer，修改state后，作为参数传入下一个(state,action)=>{}函数
	  // 不符时，直接将state作为参数传入下一个(state,action)=>{}函数
	  var reducers = (0, _keys2.default)(handlers).map(function (type) {
	    return handleAction(type, handlers[type]);
	  });
	
	  // 流程控制，按action.type执行相应的reducer，修改state；或跳过reducer执行，将state传递到下一个流程
	  var reducer = reduceReducers.apply(undefined, (0, _toConsumableArray3.default)(reducers));
	
	  // 按action.type执行相应的reducer，修改state
	  return function () {
	    var state = arguments.length > 0 && arguments[0] !== undefined ? arguments[0] : defaultState;
	    var action = arguments[1];
	    return reducer(state, action);
	  };
	}
	
	exports.default = handleActions;
	module.exports = exports["default"];
	
## 对外接口

### index.js

	'use strict';
	
	Object.defineProperty(exports, "__esModule", {
	  value: true
	});
	
	var _hashHistory = require('react-router/lib/hashHistory');
	var _hashHistory2 = _interopRequireDefault(_hashHistory);
	
	var _reactRouterRedux = require('react-router-redux');
	
	var _createDva = require('./createDva');
	var _createDva2 = _interopRequireDefault(_createDva);
	
	function _interopRequireDefault(obj) { return obj && obj.__esModule ? obj : { default: obj }; }
	
	exports.default = (0, _createDva2.default)({
	  mobile: false,
	  initialReducer: {
	    routing: _reactRouterRedux.routerReducer
	  },
	  defaultHistory: _hashHistory2.default,
	  routerMiddleware: _reactRouterRedux.routerMiddleware,
	
	  setupHistory: function setupHistory(history) {
	    this._history = (0, _reactRouterRedux.syncHistoryWithStore)(history, this._store);
	  }
	});
	module.exports = exports['default'];
	
### mobile.js

	'use strict';
	
	Object.defineProperty(exports, "__esModule", {
	  value: true
	});
	
	var _createDva = require('./createDva');
	var _createDva2 = _interopRequireDefault(_createDva);
	
	function _interopRequireDefault(obj) { return obj && obj.__esModule ? obj : { default: obj }; }
	
	exports.default = (0, _createDva2.default)({
	  mobile: true
	});
	module.exports = exports['default'];
	
## 插件引用

### 由index.js提供dva,{connect}，require("dva")实现

	Object.defineProperty(exports, "__esModule", {
	  value: true
	});
	
	exports.default = require('./lib');
	exports.connect = require('react-redux').connect;
	
### 由mobile.js提供dva,{connect}，require("dva/mobile")实现

	Object.defineProperty(exports, "__esModule", {
	  value: true
	});
	
	exports.default = require('./lib/mobile');
	exports.connect = require('react-redux').connect;
	
### 由router.js提供router,{routerRedux}，require("dva/router")实现

	module.exports = require('react-router');
	module.exports.routerRedux = require('react-router-redux');
	
### 由saga.js提供sagaMiddlerware，require("dva/saga")实现

	module.exports = require('redux-saga');

	
### 由fetch.js提供'isomorphic-fetch'模块接口，require("dva/fetch")实现

	module.exports = require('isomorphic-fetch');