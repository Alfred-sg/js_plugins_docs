# fbjs 0.8.4

## 辅助函数

### 基本

#### emptyFunction

* emptyFunction()，调用时返回undefined。
* emptyFunction.thatReturnsThis()，调用时返回this。
* emptyFunction.thatReturnsArgument(arg)，调用时返回arg。
* emptyFunction.thatReturns(arg)，创建用于返回arg的函数。
* emptyFunction.thatReturnsFalse()，创建用于返回false的函数。
* emptyFunction.thatReturnsTrue()，创建用于返回true的函数。
* emptyFunction.thatReturnsNull()，创建用于返回null的函数。

#### emptyObject

* 冻结的空对象。

#### nullthrows

* nullthrows(arg)，首参arg不为null或undefined时，直接抛出；否则报"Got unexpected null or undefined"错误。

#### sprintf

* sprintf(format)，以次参等替换首参format字符串中的%s后输出。

### 转译相关

#### shouldPolyfillES6Collection

* shouldPolyfillES6Collection("Map"|"Set")，校验浏览器平台是否需要通过babel等转译后，才能支持Map、Set类型，及该类型的next迭代、keys、forEach、clear等方法。

### 编码相关

#### base62

* base62(number)，将数值转化成base62编码字符串并输出。

#### crc32

* crc32(str)，crc32冗余校验码，用于判断写入镜像文件的数据取出时是否损坏。

## 数据转换

### 字符串

#### camelize

* camelize(string)，将字符串从连字符书写形式转化为小驼峰式书写形式。

#### hyphenate

* hyphenate(string)，将参数string从小驼峰式书写形式转化为连字符书写。

### 数组

#### compactArray

* compactArray(arr)，过滤首参数组中的null及undefined，等同(arr).filter(item=>item!=null)。

#### concatAllArray

* concatAllArray([arr1,...arrN])，将二维数组的各数组项拼接为单一数组后返回。

#### flattenArray

* flattenArray(arr)，将深度嵌套的数组arr转化为扁平化数组后输出。

#### flatMapArray

* flatMapArray(arr,fn)，遍历数组arr，执行回调fn.call(arr,arr[ii],ii)，将回调fn的返回值拼接为数组后输出。

#### createArrayFromMixed

* createArrayFromMixed(obj|arr|nodeList)，将单个对象、数组或伪数组转化成数组后输出。

#### groupArray

* groupArray(array,fn)，遍历array，执行回调fn.call(array,array[ii],ii)，以获取分组的属性名，将array打包分组。

#### partitionArray

* partitionArray(array,predicate,context)，遍历array，执行回调predicate.call(context,array[ii],ii,array)；将数组array中匹配与不匹配predicate回调的元素分成两组后以数组形式返回。

#### removeFromArray

* removeFromArray(array,element)，从数组array中移除元素element。

### 对象

#### forEachObject

* forEachObject(object,callback,context)，遍历object对象的属性，以context作为上下文执行callback函数。

#### mapObject

* mapObject(object,callback,context)，遍历object对象的属性，以context作为上下文执行callback函数。不同于forEachObject函数的无返回值，mapObject以对象形式输出回调callback执行结果。

#### everyObject

* everyObject(object,callback,context)，遍历object对象的属性，以context作为上下文执行callback函数；当callback返回否值时，遍历终止。

#### someObject

* someObject(object,callback,context)，遍历object对象的属性，以context作为上下文执行callback函数；雷同[].some(fn)，某属性在调用callback返回真值，则最终输出true；否则为false。

#### filterObject

* filterObject(object,callback,context)，遍历object对象的属性，以context作为上下文执行callback函数；雷同[].filter(fn)，callback返回真值，则最终输出对象中保留对象obj的属性。

#### partitionObject

* partitionObject(object,predicate,context)，遍历object对象的属性，以context作为上下文执行predicate函数；将匹配与不匹配predicate回调的object属性分为两组，构成数组形式后输出。

#### partitionObjectByKey

* partitionObjectByKey(source,whitelist)，借助partitionObject遍历source对象的属性，将包含在与不包含在白名单whitelist中的object属性分为两组，构成数组形式后输出。

#### keyOf

* keyOf(oneKeyObj)，返回单属性对象oneKeyObj的属性，或null。

#### keyMirror

* keyMirror(obj)，等同Object.keys(obj)获取对象的属性，却以对象形式返回对象的属性到其自身的映射。

#### keyMirrorRecursive

* keyMirrorRecursive(obj,prefix)，加前缀prefix获取对象属性到属性的映射；若为深度嵌套形式，则映射为属性到值对象映射属性对象。

#### countDistinct

* countDistinct(iter,selector)，通过selector转换迭代器iter中各迭代值后，通过Set类型后计数。
    
#### areEqual

* areEqual(a,b)，校验a、b是否完全值相等。

## DOM-BOM处理

### 能力检测

#### ExecutionEnvironment

* ExecutionEnvironment.canUseDOM，能否使用浏览器的dom操作接口。
* ExecutionEnvironment.canUseWorkers，是否webworker，浏览器启动的js线程。
* ExecutionEnvironment.canUseEventListeners，能否使用浏览器的事件接口。
* ExecutionEnvironment.canUseViewport，判断是否可以获取屏幕信息。
* ExecutionEnvironment.isInWorker，是否在webworker线程中。

### 节点操作

#### getMarkupWrap

* getMarkupWrap(nodeName)，以html方式创建节点nodeName时，获取必要的外围节点包裹新创建的节点[depth,openTags,closeTags]；无需包裹返回null。

#### containsNode

* containsNode(outerNode, innerNode)，判断innerNode是否outerNode的子节点。

#### createNodesFromMarkup

* createNodesFromMarkup(markup, handleScript)，markup以html形式设置子节点，并返回子节点node对象集合nodeList；handleScript为真值允许设置script节点，并以函数形式处理script节点。

### 样式

#### camelizeStyleName

* camelizeStyleName(style)，将连字符书写的样式名转化为小驼峰式，特别将"-ms-"起始的样式名转化成以"ms"起始。

#### hyphenateStyleName

* hyphenateStyleName(style)，将小驼峰式书写的样式名转化为连字符式，特别将"ms-"起始的样式名转化成以"-ms-"起始。

#### joinClasses

* joinClasses(className)，将多个样式字符串拼接为单样式形式。

#### cx

* cx({className1:true,className2:false})，将值为真的属性拼接为样式输出。
* cx(className1,...calssNameN)，将多个字符串拼接为样式后输出。

#### CSSCore

* addClass(element,className)，为元素element添加样式className。
* removeClass(element,className)，为元素element移除样式className。
* conditionClass(element,className,bool)，按条件bool的真假，为元素element添加或移除样式className。
* hasClass(element,className)，判断元素element是否包含样式className。
* matchesSelector(element,selector)，判断元素element是否匹配选择器selector。









