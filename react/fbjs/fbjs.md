# fbjs 0.8.4

## 辅助函数

### emptyFunction

* emptyFunction()，调用时返回undefined。
* emptyFunction.thatReturnsThis()，调用时返回this。
* emptyFunction.thatReturnsArgument(arg)，调用时返回arg。
* emptyFunction.thatReturns(arg)，创建用于返回arg的函数。
* emptyFunction.thatReturnsFalse()，创建用于返回false的函数。
* emptyFunction.thatReturnsTrue()，创建用于返回true的函数。
* emptyFunction.thatReturnsNull()，创建用于返回null的函数。

### emptyObject

* 冻结的空对象。

### shouldPolyfillES6Collection

* shouldPolyfillES6Collection("Map"|"Set")，校验浏览器平台是否需要通过babel等转译后，才能支持Map、Set类型，及该类型的next迭代、keys、forEach、clear等方法。

### base62

* base62(number)，将数值转化成base62编码字符串并输出。

### crc32

* crc32(str)，crc32冗余校验码，用于判断写入镜像文件的数据取出时是否损坏。

## 数据转换

### camelize

* camelize(string)，将字符串从连字符书写形式转化为小驼峰式书写形式。

### compactArray

* compactArray(arr)，过滤首参数组中的null及undefined，等同(arr).filter(item=>item!=null)。

### concatAllArray

* concatAllArray([arr1,...arrN])，将二维数组的各数组项拼接为单一数组后返回。

### countDistinct

* countDistinct(iter,selector)，通过selector转换迭代器iter中各迭代值后，通过Set类型后计数。

### createArrayFromMixed

* createArrayFromMixed(obj|arr|nodeList)，将单个对象、数组或伪数组转化成数组后输出。
    
### areEqual

* areEqual(a,b)，校验a、b是否完全值相等。

## DOM处理

### 能力检测

#### ExecutionEnvironment

* ExecutionEnvironment.canUseDOM，能否使用浏览器的dom操作接口。
* ExecutionEnvironment.canUseWorkers，是否webworker，浏览器启动的js线程。
* ExecutionEnvironment.canUseEventListeners，能否使用浏览器的事件接口。
* ExecutionEnvironment.canUseViewport，判断是否可以获取屏幕信息。
* ExecutionEnvironment.isInWorker，是否在webworker线程中。

#### camelizeStyleName

* camelizeStyleName(style)，将连字符书写的样式名转化为小驼峰式，特别将"-ms-"起始的样式名转化成以"ms"起始。

#### containsNode

* containsNode(outerNode, innerNode)，判断innerNode是否outerNode的子节点。









