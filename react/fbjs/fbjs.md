# fbjs 0.8.4

## 辅助函数

### 基本

#### invariant

* invariant(condition,format,a,b,c,d,e,f)，condition为否值时，以处理"%s"字符后的format字符串信息报错。

#### warning

* warning(condition,format)，非production环境下，condition为否值时，以处理"%s"字符后的format字符串信息报错，以查找堆栈信息；production环境下，不作处理。

#### memoizeStringOnly

* memoizeStringOnly(callback)，返回(key)=>{}，调用callback回调设置cache[key]闭包缓存、或者获取cache[key]闭包缓存。

#### xhrSimpleDataSerializer

* xhrSimpleDataSerializer(data)，将数组或对象data转换为查询字符串输出，如key=value；实际查询字符串需要data[key]=value。

#### URI

* new URI(uri)，创建URI实例，toString方法获取uri。

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

#### setImmediate

* setImmediate(callback,...args)，等待强业务逻辑函数执行完成后，运行callback回调。

#### resolveImmediate

* resolveImmediate(callback)，通过promise机制执行callback回调函数。

#### ErrorUtils

* 调用函数报错时供抛出堆栈使用。


### 转译相关

#### shouldPolyfillES6Collection

* shouldPolyfillES6Collection("Map"|"Set")，校验浏览器平台是否需要通过babel等转译后，才能支持Map、Set类型，及该类型的next迭代、keys、forEach、clear等方法。


### 异步处理

#### Promise

* new Promise((resolve,reject)=>{})，调用promise模块构建延迟对象，含node端方法。

#### promise.native

* new Promise((resolve,reject)=>{})，调用promise模块构建延迟对象，只包含浏览器端方法。

#### Deferred

* new Deferred()，以jquery方式处理异步逻辑。

#### PromiseMap

* new PromiseMap()，以key键构建多个deferred实例，处理异步逻辑。


### 版本号

#### VersionRange

* VersionRange.contains(range,version)，判断实际版本号version是否在range范围中。


### 编码相关

#### base62

* base62(number)，将数值转化成base62编码字符串并输出。

#### crc32

* crc32(str)，crc32冗余校验码，用于判断写入镜像文件的数据取出时是否损坏。

#### TokenizeUtil

* TokenizeUtil.getPunctuation()，获取标点符号的匹配正则。

#### UnicodeUtils

* UnicodeUtils.getCodePoints(str)，从字符串转化成unicode字符集数组。
* UnicodeUtils.getUTF16Length(str,pos)，BMP字符集[U+0000..U+D7FF]或[U+E000, U+FFFF]返回1，非BMP字符集[U+D800..U+DFFF]返回2。
* UnicodeUtils.hasSurrogateUnit(str)，判断是否含有非BMP字符集。
* UnicodeUtils.isCodeUnitInSurrogateRange(codeUnit)，判断字符codeUnit是否非BMP字符集[U+D800..U+DFFF]。
* UnicodeUtils.isSurrogatePair(str,index)，判断str[index]是否UTF16编码代理对，以SurrogatePair起始。
* UnicodeUtils.strlen(str)，BMP字符集直接返回长度，非BMP字符集两位起跳。
* UnicodeUtils.substring(str,start,end)，BMP字符集直接以参数截取字符，非BMP字符集两位起跳。
* UnicodeUtils.substr(str,start,length)， BMP字符集直接以参数截取字符，非BMP字符集两位起跳。

#### UnicodeUtilsExtra

* formatCodePoint(codePoint)，Unicode字符输出，U+XXXX、U+XXXXX、U+XXXXXX形式。
* getCodePointsFormatted(str)，将字符串转化成unicode字符集数组，高位缺失则补充。
* zeroPaddedHex(codePoint,len)，Unicode字符高位补充，"1"补充为"0001"。
* phpEscape(str)、jsEscape(str)、cEscape(str)、objcEscape(str)、pyEscape(str)，转化成特定语言的字符串形式。

#### UnicodeBidiDirection

* NEUTRAL、LTR、RTL，获取"NEUTRAL"、"LTR"、"RTL"文本取向字符串。
* isStrong(dir)，判断dir是否"LTR"、"RTL"中的一个。
* getHTMLDir(dir)，获取小写形式的"ltr"或"rtl"。
* getHTMLDirIfDifferent(dir,otherDir)，当dir不等于otherDir时，以dir转化获取"ltr"或"rtl"。
* setGlobalDir、getGlobalDir、initGlobalDir设置、获取或初始化设置缓存变量globalDir，初始化默认值为"ltr"。

#### UnicodeBidi

缺省。

#### UnicodeBidiService

缺省。

#### UnicodeCJK

缺省。

#### UnicodeHangulKorean

缺省。



## 数据交互

### 发送请求

#### fetch

* fetch(url, options)，调用node-fetch模块实现node端发送请求；调用whatwg-fetch实现客户端发送ajax请求。

#### fetchWithRetries

* fetchWithRetries(url,{fetchTimeout,retryDelays})请求失败或超时后，尝试重新发送请求。参数fetchTimeout设定超时时间，超时尝试重新发送请求；参数retryDelays的长度限定尝试发送请求的次数，值限定尝试请求与上一次请求的时间间隔。

### 媒体文件类型

#### PhotosMimeType

* PhotosMimeType.isImage|isJpeg(mimetype)，用于判断媒体文件类型是否图片。

#### DataTransfer

* new DataTransfer(data)，资源类型判断或或获取资源。



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

#### distinctArray

* distinctArray(arr)，通过arr类型将xs数组去重后返回。


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


### Set类型

#### Set

* 利用'core-js'模块构建Set类型。

#### someSet

* someSet(set,predicate,context)，遍历set，当有成员匹配predicate.bind(context)时，返回真值；否则返回否值。

#### everySet

* everySet(set,predicate,context)，遍历set，当所有成员均匹配predicate.bind(context)时，返回真值；否则返回否值。

#### equalsSet

* equalsSet(a,b)，比较两个set实例a、b是否等值。

### Map类型 

#### Map

* new Map()，利用'core-js'模块构建Map类型。


### interator迭代器

#### enumerate

* enumerate["KIND_KEYS"|"KIND_VALUES"|"KIND_ENTRIES"]，获取迭代类型常数"keys"|"values"|"entries"。
* enumerate.keys(data)，获取键迭代器；字符串无，数组序号，对象键，或默认迭代器。
* enumerate.values(data)，获取值迭代器；字符串字符，数组项，对象值，或默认迭代器。
* enumerate.entries(data)，获取键、值迭代器；字符串无，数组序号加元素，对象键加值，或默认迭代器。
* enumerate.generic(object)，获取对象的属性、值迭代器。

#### equalsIterable

* equalsIterable(iter1,iter2,areEqual=(item1,item2)=>{})，通过areEqual判断两个迭代器的数据是否相同；默认判断各迭代值相同，可通过areEqual改写比较逻辑。

#### countDistinct

* countDistinct(iter,selector)，通过selector转换迭代器iter中各迭代值后，通过Set类型后以去重形式计数。


### 判断比较

#### isEmpty

* isEmpty(object|array|...)，用于判断对象、数组为空，非普通数据类型非否值；不能判断迭代器。

#### minBy

* minBy(iterator,transform,compare)，遍历迭代器iterator，将迭代值传入transform转换函数中，以获取比较值，通过比较函数compare作比较，输出最小比较值对应的迭代值。compare函数默认为数值比较。

#### maxBy

* maxBy(iterator,transform,compare)，基于minBy，设置compare，输出最大比较值对应的迭代值。compare函数默认为数值比较。

#### shallowEqual

* shallowEqual(a,b)，当a、b为普通数据类型时，比较值相等；为数组或对象，比较数组长度、属性个数是否相等，通过"==="浅比较首层数组项是否相等。
    
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

#### performance

* 通过window.performance作性能检测用。

#### performanceNow

* performanceNow()，获取页面开始加载到js代码执行时的时间戳，可用于计算代码执行的耗时。

#### UserAgentData

* UserAgentData.browserArchitecture | browserFullVersion | browserMinorVersion | browserName | browserVersion | deviceName | engineName | engineVersion | platformArchitecture | platformName | platformVersion | platformFullVersion，通过ua-parser-js模块解析window.navigator.userAgent，获取系统、设备、浏览器、cpu、引擎等相关数据。

#### UserAgent

* UserAgent.isBrowser | isBrowserArchitecture | isDevice | isEngine | isPlatform | isPlatformArchitecture，校验浏览器版本等信息是否匹配特定条件。


### 节点操作

#### getMarkupWrap

* getMarkupWrap(nodeName)，以html方式创建节点nodeName时，获取必要的外围节点包裹新创建的节点[depth,openTags,closeTags]；无需包裹返回null。

#### containsNode

* containsNode(outerNode, innerNode)，判断innerNode是否outerNode的子节点。

#### createNodesFromMarkup

* createNodesFromMarkup(markup, handleScript)，markup以html形式设置子节点，并返回子节点node对象集合nodeList；handleScript为真值允许设置script节点，并以函数形式处理script节点。

#### focusNode

* focusNode(node)，使node节点获得焦点。

#### getActiveElement

* getActiveElement()，输出页面中获得焦点的元素，或者document.body，或者null。

#### isNode

* isNode(node)，判断node是否dom节点。

#### isTextNode

* isTextNode(node)，基于isNode函数，判断node是否文本节点。


### 样式

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

#### camelizeStyleName

* camelizeStyleName(style)，将连字符书写的样式名转化为小驼峰式，特别将"-ms-"起始的样式名转化成以"ms"起始。

#### hyphenateStyleName

* hyphenateStyleName(style)，将小驼峰式书写的样式名转化为连字符式，特别将"ms-"起始的样式名转化成以"-ms-"起始。

#### getStyleProperty

* getStyleProperty(node,styleName)，获取node节点的styleName样式。

#### Style

* Style.get(node,styleName)，获取node节点的styleName样式。
* Style.getScrollParent(node)，向上获取首个滚动的父元素，包含其自身。

#### Scroll

* Scroll.getTop(node)，获取上下滚动偏移量。
* Scroll.setTop(node)，设置上下滚动偏移量。
* Scroll.getLeft(node)，获取左右滚动偏移量。
* Scroll.setLeft(node)，设置左右滚动偏移量。

#### getDocumentScrollElement

* getDocumentScrollElement()，为滚动取值获取文档节点；getScrollPosition函数中使用。

#### getUnboundedScrollPosition

* getUnboundedScrollPosition(scrollable)，获取scrollable滚动节点的偏移量或window对象的pageOffset属性，以{x,y}输出；getScrollPosition模块中使用。

#### getScrollPosition

* getScrollPosition(scrollable)，获取滚动节点scrollable的偏移量，{x,y}形式输出。

#### getElementRect

* getElementRect(elem)，获取elem节点四周边际离页面上、左边际的距离，{top,bottom,left,right}形式。

#### getElementPosition

* getElementPosition(elem)，获取elem节点距离页面上、左边际的距离，以及节点的宽、高。

#### getViewportDimensions

* getViewportDimensions()，获取页面的宽高，包含滚动条。
* getViewportDimensions.withoutScrollbars()，获取页面的宽高，不包含滚动条。

#### nativeRequestAnimationFrame

* nativeRequestAnimationFrame(cb)，cb中操纵dom节点实现动画，借助window.requestAnimationFrame实现。

#### requestAnimationFrame

* requestAnimationFrame(cb)，当浏览器不支持window.requestAnimationFrame，通过setTimeout实现延迟执行回调cb，以实现动画。



### 事件

#### EventListener

* EventListener.listen(target,eventType,callback)，非捕获模式绑定事件，返回值用于解绑事件。
* EventListener.capture(target,eventType,callback)，捕获模式绑定事件，返回值用于解绑事件，IE不支持捕获模式。
* EventListener.registerDefault()，空操作。

#### monitorCodeUse

* monitorCodeUse(eventName,data)，检测eventName只包含[^a-z0-9_]字符。

#### Keys

* 通过键盘按键获取对应的键盘码，如Keys.ESC=27。

#### TouchEventUtils

* TouchEventUtils.extractSingleTouch(nativeEvent)，获取触碰事件touchObj对象，以得到PageX等属性。









