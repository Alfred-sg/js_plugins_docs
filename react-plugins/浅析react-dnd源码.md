# 浅析react-dnd源码 @2.4.0

## 思想实验

假使需要设计一个由无人机派送快递的指挥系统，当一架无人机开始起飞时，它将信号发送至指挥中心，由指挥中心记录该无人机的飞行状态，如位置、时速、机体状况等，并结合无人机的飞行高度、天气状况等信息，给无人机发送反馈信号，以影响无人机的飞行；当无人机到达某个派送位时，指挥中心发送的信号又能促使该无人机减速降落；等到客户取走快递后，指挥中心的信号又能促使该无人机再次起飞；当无人机送完快递返航后，指挥中心又将取消追踪该无人机的飞行状态。

首先，指挥中心（Manager）担纲着接受信号receive_message、处理信号handle_message、发送命令send_commander三种职责。而无人机（UVA）发送的信号则包含起飞fly_start、在特定区域飞行中fly_over、进入特定区域fly_enter、离开特定区域fly_leave、派送包裹中fly_drop、返回停靠站fly_end这几种，附带位置信息address_data、机体状况status_data。

假使无人机存在性能差异，指挥中心在决定无人机的飞行速度时会采用不同的策略，那么指挥中心就有必要记录无人机的id（uvaId），通过该id委派任务。当无人机在出行过程中，另有客户在系统中提交了派送邮件的流程，指挥中心就需要知道无人机实际的载重信息和飞行路径，当其出行路径覆盖客户所在地、且其载重可接受客户的包裹时，就对该无人机下达接受包裹的命令，不然，需指派另一架无人机完成该任务。

同时，指挥中心又会把无人机即将抵达的消息发送给客户，为着引发一系列副作用，如命令无人机筛选出客户的包裹、在订单系统中将该客户的包裹标注为已派送等，指挥中心就有必要记录该客户的id（targetId）。甚至于，在无人机出行过程中，可根据飞行区域的不同，对定居人群投放特定形式的语音广告，或者由特定的驿站存放包裹。若是仍以targetId标记这部分人群和驿站，那么targetId就可以订阅无人机的fly_over、fly_enter、fly_leave、fly_drop事件，相应地触发一系列动作，如接受包裹、收听语音广播等。

再设想无人机由不同厂商生产，它们的启动脚本不尽相同。我们可以抽象出一个UVA适配器类，利用它来执行各种款式型号的无人机启动脚本。虽然可以在这个类中通过判断无人机的类型执行特定的语法指令，但就代码的清晰度、可扩展性考虑，我们不如将这个适配器类注入到无人机的启动脚本里，促使它在启动过程加载某些配置项，以使无人机在起飞、降落过程中执行一系列副作用，比如计算载重信息。

对于客户，指挥系统实际上无法对他们直接下达指令，但是指挥系统可以通知客户无人机即将抵达，让他们提前做好接受包裹的准备。因此，我们可以再抽象一个Target类，当其订阅fly_drop事件时，就能通过该抽象类实现向客户发送无人机即将抵达的消息、或者在订单系统中将客户订单标注为已派送的功能。甚至于，我们可以在这个Target类中实现canDrop方法，当客户无意在这时候接受邮件时，指挥中心就会告知无人机需要继续飞往下一个投放点。

如果我们再将redux状态管理机制引入系统中。当无人机执行飞行动作、触发fly_start事件时，使用store.dispatch方法派发一个action。在这个action执行过程中，首先判断无人机是否可以起飞canFly，其次便是执行UVA适配器类的startFly方法，用于获取无人机的实际载重信息或者执行订制任务，再将无人机的id、载重信息等存入store.state中。当无人机经过特定区域或者某个客户所在地时，我们也将利用store.state存储targetId，继而通过Target类通知客户无人机已经抵达。当无人机返仓时，我们又将通过redux促使UVA适配器类执行endFly方法，这样，就可以为年久失修的无人机订制一个维修服务。

通过以上说法，UVA适配器类独立于无人机的启动、降落脚本，可视为无人机启动、降落时须执行的附加命令，同时可对不同的无人机配置特定的UVA适配器类。

react-dnd相较这一思想实验，dnd-core相当于以action-state状态管理机制更新系统中无人机飞行状况、载重信息、订单系统数据等状态值；react-dnd相当于为特定无人机注册特定的钩子函数，协调指挥系统到无人机的起停命令；react-dnd-html5-backend相当于在无人机执行起停命令时通过事件机制执行通用脚本、和派发action执行订制脚本。

## 整体构造

dnd-core 使用redux管理拖拽状态，注册DragSource,DropTarget钩子函数集合；对外提供monitor获取state数据，getActions执行action，执行action的同时会调用DragSource,DropTarget中的钩子。

react-dnd 装饰拖拽组件DragSource、投放位组件DropTarget，为被装饰组件注入props；props中包含connectDragSource, connectDragPreview, connectDropTarget，用于为实际的dom节点绑定拖拽事件。

react-dnd-html5-backend 为实际的dom节点绑定拖拽事件，调用action，更新state，计算offset。可装卸。

## dnd-core包

### 工作流程
          
                           +--------------------------------------------------+
                           |                 DragDropManager                  |
                           |                                                  |
   DragSource/DropTarget   |  +----------+                       +---------+  |
  ------------------------>|->| registry |-------+   +-----------| monitor |  |
                           |  +----------+       |   |           +---------+  |
                           |       |             |   v                ∧       |  
                           |       |             | hooks:             |       |   
            +--------------|-------+             | DragSource         |       |
            |              |                     | DropTarget         |state  |
            |              |                     |                    |       |
            |              |  +------------+     v                +-------+   |
            | sourceId     |  | getActions |--------------------->| store |   |
            | targetId     |  +------------+     update state     +-------+   |
            |              |       ∧                                          |
            |              |       |                                          |
            |              +--------------------------------------------------+
            |                      |                
            v                      |                
   action when drag or drop   +---------+           
  ----------------------------| actions |
                              +---------+

1. 在容器组件中创建DragDropManager实例，在其实例化过程中创建redux.store，并将其注入为子组件的context，子组件可通过DragDropManager实例获取redux的state状态，以及可执行用以改变state的actions。

2. 当子组件实例化或更新子组件时，将DragSource或DropTarget特定钩子函数集合缓存在HandlerRegistry实例的handlers属性中。其意义是，特定actions执行过程中，将通过DragSource或DropTarget的id获得特定的handler并执行。特别的，在首个可拖拽组件实例化过程中，将调用backend的setup方法为window对象绑定监听拖拽事件的方法。

3. 当组件拖拽、悬停、投放到某位置或拖拽结束时，通过dragDropManager.getActions()中各方法执行特定的action，改变state状态的同时，将执行拖拽组件关联的DragSource或DropTarget钩子函数。

4. 子组件beginDrag时，通过执行handlerRegistry.pinSource方法将其对应DragSource存入handlerRegistry.pinnedSource中；当子组件endDrag时，通过handlerRegistry.unpinSource方法置空handlerRegistry.pinnedSource。

5. 在子组件拖拽过程中，若想获得redux的state状态，可通过dragDropManager.monitor对象的方法获得相关数据。

6. 当组件卸载时，从handlerRegistry中移除DragSource或DropTarget特定钩子函数集合；当所有可拖拽组件均已卸载，通过订阅state.refCount的变更，执行backend的teardown方法进行销毁。

### 主要模块

1. DragDropManager构造函数，在其实例化过程中将创建redux.stroe，实例化DragDropMonitor并存储为this.monitor属性，对外输出this.store[即redux.store], this.monitor[用于获取state状态], this.registry[用于注册、移除某组件特定的DragSource或DropTarget钩子函数集合], this.getActions()[用于获取redux的actions并执行，以改变state状态], this.context[即原始由父组件注入子组件的context对象], this.backend[装载的拖拽事件监听策略]。

DragDropManager实例同时将通过store.subscribe方法监听state.refCount变更。当首个可拖拽组件被渲染到视图上时，执行backend的setup方法进行初始化，在react-dnd-html5-backend中表现为window对象绑定监听函数；当所有可拖拽组件被移除时，执行backend的teardown方法进行销毁，在react-dnd-html5-backend中表现移除window对象绑定的监听函数。

2. HandlerRegister构造函数，用于注册、移除或获取某组件特定的DragSource或DropTarget钩子函数集合。

HandlerRegister实例对外输出this.addSource(type,source)[注册DragSource，返回sourceId], this.getSource[获取DragSource], this.getSourceType[获取某拖拽组件相应的type], this.isSourceId[判断是否DragSource], this.removeSource[移除DragSource], this.addTarget[注册DropTarget，返回targetId], this.getTarget[获取DropTarget], this.getTargetType[获取某投放位组件相应的type], this.isTargetId[判断是否DropTarget], this.removeTarget[移除DropTarget], this.addHandler(role,type,handler)[addSource | addTarget方法执行时调用，用于添加this.handlers及this.types], this.containsHandler(DragSource | DropTarget)[判断某个DragSource或DropTarget是否已注册], this.pinSource(sourceId)[某可拖拽组件被拖拽时，使用this.pinnedSource记录其对应的钩子函数集合DragSource], this.unpinSource[某可拖拽组件停止拖拽时，置空this.pinnedSource]。

在DragSource或DropTarget注册过程中，state.refCount加1，并将DragSource或DropTarget添加到this.handlers数组中，this.types对象中添加{handlerId: type}；移除时，state.refCount减1，从this.handlers数组移除DragSource或DropTarget，并移除this.types[handlerId]。

DragSource或DropTarget钩子函数集合通过role = "SOURCE" | "TARGET"作区分。

3. actions/*.js设定actionCreators[actionCreators的上下文为DragDropManager实例]，reducers/*.js设定reducers。

actions/registry.js中，含有addSource, removeSource, addTarget, removeTarget四个actionCreator。相应的reducer为reducers/refCount.js，促使state.refCount增1或减1；reducers/dirtyHandlerIds.js，将state.dirtyHandlerIds置为NONE = []。

actions/dragDrop.js中，含有beginDrag, publishDragSource, hover, drop, endDrag四个actionCreator。

  * beginDrag(sourceIds,options={publishSource,clientOffset,getSourceClientOffset})，首先获取拖拽组件的sourceId[即uuid，HandlerRegister实例执行addSource方法时生成]；通过registry.getSource(sourceId)获取对应的DragSource；执行DragSource的beginDrag钩子，以获取item；通过registry.pinSource(sourceId)将DragSource存入registry.pinnedSource中；通过registry.getSourceType获取itemType。最终生成action = { type: BEGIN_DRAG, itemType, item, sourceId, clientOffset, sourceClientOffset: getSourceClientOffset(sourceId), isSourcePublic: publishSource }。

  在reducers/dragOperation.js中，更新state.dragOperation的itemType, item, sourceId, isSourcePublic,  dropResult:null, didDrop: false属性。

  在reducers/dragOffset.js中，更新state.dragOffset的initialSourceClientOffset: action.sourceClientOffset, initialClientOffset: action.clientOffset, clientOffset属性。

  在reducers/dirtyHandlerIds.js中，更新state.dirtyHandlerIds更新为ALL = []。

  * publishDragSource()，当确实有组件被拖拽时，生成action = { type: PUBLISH_DRAG_SOURCE }。

  在reducers/dragOperation.js中，更新state.dragOperation的isSourcePublic属性为真。

  在reducers/dirtyHandlerIds.js中，更新state.dirtyHandlerIds更新为ALL = []。

  * hover(targetIdsArg,{clientOffset})，首先通过monitor.getItemType()获取被拖拽组件beginDrag钩子执行时存入state.dragOperation中的itemType属性，其次使用该ItemType剔除targetIdsArg可悬停组件中未匹配的元素项；遍历剩余的target，执行target的hover方法，产生如样式拖拽节点悬停时样式变更等副作用。最终生成action = { type: HOVER, targetIds, clientOffset }。

  在reducers/dragOperation.js中，更新state.dragOperation的targetIds属性。

  在reducers/dragOffset.js中，更新state.dragOffset的clientOffset属性。

  在reducers/dirtyHandlerIds.js中，更新state.dirtyHandlerIds更新为prevState.dragOperation.targetIds和action.targetIds的差集。

  * drop(options={})，首先通过monitor.getTargetIds获得actions.drop方法执行时注入的state.dragOperation.targetIds；其次通过monitor.canDropOnTarget过滤targetId；逆序遍历target，执行target的drop方法，其返回值构成action = { type: DROP, dropResult: { ...options, ...dropResult} }。注：遍历每一个target均派发action。

  在reducers/dragOperation.js中，更新state.dragOperation的dropResult, didDrop:true, targetIds:[]属性。

  在reducers/dragOffset.js中，更新state.dragOffset的initialSourceClientOffset:null, initialClientOffset:null, clientOffset:null属性。

  在reducers/dirtyHandlerIds.js中，更新state.dirtyHandlerIds更新为ALL = []。

  * endDrag()，执行DragSource的endDrag方法；执行registry.unpinSource置空registry.pinnedSource。最终生成action = { type: END_DRAG }。

  在reducers/dragOperation.js中，更新state.dragOperation的itemType:null, item:null, sourceId:null, dropResult:null, didDrop:false, isSourcePublic:null, targetIds:[]属性。

  在reducers/dragOffset.js中，更新state.dragOffset的initialSourceClientOffset:null, initialClientOffset:null, clientOffset:null属性。

  在reducers/dirtyHandlerIds.js中，更新state.dirtyHandlerIds更新为ALL = []。

4. DragDropMonitor构造函数，作为DragDropManager实例的monitor属性，用于获取redux/state状态值。

DragDropMonitor实例对外输出this.subscribeToStateChange(listener,handlers)[通常是，state.stateId变更且无组件在拖拽过程中执行linstener函数，通过store.subscribe方法监听state变更], this.subscribeToOffsetChange(linster)[当state.dragOffset变更时，执行listener函数], canDragSource(sourceId)[判断组件是否可被拖拽], canDropOnTarget(targetId)[判断组件是否可投放，须先比对被拖拽组件的type是否与target的type匹配，其次调用target.canDrop方法加以判断], isDragging()[通过this.getItemType方法判断是否有组件被拖拽过程中], isDraggingSource(sourceId)[判断DragSource是否在拖拽过程中], isOverTarget(targetId,{shallow})[判断被拖拽组件是否悬停在target组件上；shallow为真，target须是悬停位首个可dropable的组件], getItemType, getItem, getSourceId, getTargetIds, getDropResult, didDrop, isSourcePublic[获取state.dragOperation状态数据], getInitialClientOffset, getInitialSourceClientOffset, getClientOffset, getSourceClientOffset, getDifferenceFromInitialOffset[获取state.dragOffset状态数据或由此产生的计算值]。

5. DragSource接口，作为可拖拽组件的钩子函数集合，实现时须包含canDrag, isDragging, endDrag方法，同时可实现beginDrag方法。当组件内的dom节点绑定拖拽事件时，通过执行canDag方法校验该节点是否可拖拽；当组件被拖拽时，通过执行beginDrag方法获取注入到state.dragOperation中的item属性；当组件结束拖拽时，执行endDrag方法，以产生副作用。isDragging(monitor,sourceId)方法用于判断组件是否在拖拽过程中。

DropTarget接口，作为可投放位组件的钩子函数集合，实现时须包含canDrop, hover, drop方法。当被拖拽组件悬停在target上时，执行hover方法，以产生副作用；当组件投放在target上时，获取dropResult，执行drop方法，并注入到state.dragOperation中。通过canDrop判断target组件所在位置是否可投放。

6. backends/createTestBackend.js，模拟backend，即dom节点绑定拖拽事件的策略；测试用。

### 关于设计

DragSource、DropTarget为特定组件设定拖拽、投放时的钩子函数，其意义是将监听拖拽事件的绑定函数与action解耦，action只与组件关联。若然在拖拽节点的绑定事件中调用boundAction生成不同组件的action，backend将与可灵活配置的组件强关联，backend也就无法作为可装载卸载的插件了。

### 使用

  import { DragDropManager, DragSource, DropTarget, createTestBackend } from "react-dnd";

  class CustomDragSource extends DragSource{
    canDrag(){}
    beginDrag(){}
    isDragging(){}
    DragSource(){}
  };

  class CustomDropTarget extends DropTarget{
    canDrop(){}
    hover(){}
    drop(){}
  };

  const manager = new DragDropManager(createTestBackend);// 作为全局缓存

  const TYPE = "TYPE";
  const source = new CustomDragSource();
  const target = new CustomDropTarget();

  // 组件被渲染到视图中时完成注册，亦可适用于jquery等框架
  const sourceId = manager.registry.addSource(TYPE,source);
  const targetId = manager.registry.addTarget(TYPE,target);

  // ...绑定拖拽事件
  node.addEventListener("hover",()=>{
    manager.getActions().hover();
  })

  // 当组件从视图中移除时
  manager.registry.removeSource(sourceId);
  manager.registry.removeTarget(sourceId);

## react-dnd包

### 工作流程

                           +-----------------------------------------+                 
          drop,hover...    |                                         |     
  spec---------------------|-------------------------------+         |              
                           |                               |         |
                           |  +------------------+         |         |
      +--------------------|--| handlerContector |         |         |
      | connectDragSource  |  +------------------+         v         |
      | connectDragPreview |                         +------------+  |
      |                    |          +------------->| DropSource |  |
      |                    |          |              +------------+  |
      |     collect        |  +----------------+           |         |
      |   +----------------|--| handlerMonitor |           |         |
      |   | state          |  +----------------+           |         |
      v   v                |                               |         |<----------------+
  +---------------+        |                               |         |                 |
  | DragComponent |------->|    DecoratedDropComponent     |         |                 |
  +---------------+        +-----------------------------------------+                 |
                                      ∧                    |                           |
                                      |                    |                           |
  +------------------+                |                    |                           |
  | DragDropContext  |        +-----------------+          |                  +------------------+
  |                  |------->| DragDropManager |<---------+                  | decoratorHandler |
  | DragDropProvider |        +-----------------+          |                  +------------------+
  +------------------+                |                    |                           |
                                      |                    |                           |
                                      v                    |                           |
  +---------------+        +-----------------------------------------+                 |
  | DragComponent |------->|    DecoratedDragComponent     |         |                 |
  +---------------+        |                               |         |                 |
      ∧   ∧                |                               |         |<----------------+
      |   | state          |  +----------------+           |         |
      |   +----------------|--| handlerMonitor |           |         |
      |     collect        |  +----------------+           |         |
      |                    |          |              +------------+  |
      |                    |          +------------->| DragSource |  |
      | connectDragSource  |                         +------------+  |
      | connectDragPreview |  +------------------+         ∧         |
      +--------------------|--| handlerContector |         |         |
                           |  +------------------+         |         |
        beginDrag...       |                               |         |
  spec---------------------|-------------------------------+         |
                           |                                         |
                           +-----------------------------------------+


1. 通过DragDropContext(backendOrModule)(DecoratedComponent)装饰父级组件，为被装饰的子组件注入this.context.dragDropManager。或者通过DragDropContextProvider构建容器组件，为子组件注入context.dragDropManager。

2. 通过DragSource(type,spec,collect,options={})(DecoratedComponent)装饰被拖拽组件DragComponent；将spec转化成dragDropManager.registry可接受DragSource并完成注册；collect用于获取注入DragComponent的props，包含为dom节点绑定拖拽事件的handlerConnector.hooks={ dragSource, dragPreview }，以及通过handlerMonitor获得的redux/state状态。

3. 被拖拽组件DragComponent内部调用handlerConnector.hooks下各方法，为实际的拖拽节点、拖拽预览节点绑定拖拽事件，绑定函数中触发redux/actions的执行。

4. 通过DropTarget(type,spec,collect,options={})(DecoratedComponent)装饰投放位组件DropComponent；将spec转化成dragDropManager.registry可接受DropTarget并完成注册；collect用于获取注入DropComponent的props，包含为dom节点绑定拖拽事件的handlerConnector.hooks={ dropTarget }，以及通过handlerMonitor获得的redux/state状态。

5. 投放位组件DropComponent内部调用handlerConnector.hooks下各方法，为实际的投放位节点绑定拖拽事件，绑定函数中触发redux/actions的执行。

### 主要模块

1. DragDropContext(backendOrModule)(DecoratedComponent)，装载拖拽事件插件backendOrModule，通过容器组件为DecoratedComponent注入this.context.dragDropManager。

经DragDropContext装饰后的组件含有原型方法getDecoratedComponentInstance[获取被装饰的组件实例], getManager[获取传入子组件的this.context.dragDropManager]。

2. DragDropContextProvider，作为容器组件，将为子组件为注入this.context.dragDropManager。

DragDropContextProvider组件含有实例属性backend，即加载的拖拽事件绑定策略。

3. decorateHandler({DecoratedComponent,createHandler,createMonitor,createConnector,registerHandler,containerDisplayName,getType,collect,options={arePropsEqual}})装饰拖拽组件DecoratedComponent。

  * 首先通过this.context.dragDropManager获取DragDropManager实例。

  * 其次通过该DragDropManager实例生成为拖拽组件DragComponent、投放位组件DropComponent服务的this.handlerMonitor，分别调用createSourceMonitor(manager), createTargetMonitor(manager)函数。handlerMonitor的意义，一是传入DragSource或DropTarget中，用于构建当前组件所对应的钩子函数集合handler；二是作为collect函数的参数，并由此生成装饰后组件的this.state，且作为被装饰组件DecoratedComponent的props。

  * 其次通过DragDropManager实例的backend属性生成this.handlerConnector。该handlerConnector属性用于为实际ReactDomComponent组件的相应dom节点绑定拖拽事件，在监听函数中触发action的执行，并更新state。为dom节点绑定监听函数过程，是通过collect函数将handlerConnector中的dragSource, dragPreview, dropTarget方法注入为装饰后组件的state，再注入为被装饰组件DecoratedComponent的props；开发者通过调用props中的dragSource, dragPreview, dropTarget方法实现为dom节点绑定拖拽事件的监听函数。

  * 其次通过handlerMonitor基于createSourceFactory, createTargetFactory生成可注册的this.handler，即DragSource, DropTarget接口的实现，可使用dragDropManager.registry.[ "addSource" | "addTarget" ]方法注入到DragDropManager实例中。

  * 在组件实例化、didMount、willReceiveProps的时候，通过执行this.receiveProps(this.props)方法更新this.handler中的props；同时将this.hander即DragSource, DropTarget注入到dragDropManager.registry中；获得handlerId用于更新this.handlerMonitor, this.handlerConnector中的handlerId；调用dragDropManager.monitor.subscribeToStateChange方法用于以执行组件的this.handleChange方法，通过this.getCurrentState方法调动collect(this.handlerConnector.hooks this.handlerMonitor)函数，以获得组件最新的state。
  
  另外，this.handleChange方法在组件didMount、willReceiveProps的时候均有执行，以更新state。

  * 在组件render过程中，将this.state, this.props作为props传入被装饰组件中DecoratedComponent，同时调用ref引用方法this.handleChildRef将被装饰组件注入this.handler，以更新DragSource, DropTarget各钩子函数的参数component。

4. DragSource(type,spec,collect,options={arePropsEqual})(DragComponent)，返回函数即拖拽组件DragComponent的装饰函数，通过调用decorateHandler模块完成装饰。

  * 参数type以常量约定拖拽组件的类型，可放置到type值相同的投放位组件DropComponent上。
  
  * spec = { canDrag(porps,monitor), beginDrag(props,monitor,component), isDragging(props,monitor,component), endDrag(props,monitor,component) }，交由用户灵活配置的钩子函数。参数monitor在装饰后组件实例化过程中生成；参数component在被装饰组件挂载时生成；参数props在装饰后组件实例化、didMount、willReceiveProps时更新。

  * collect(handlerConnector.hooks,handlerMonitor)用于获取装饰后组件的state，注入为被装饰组件的props。首参handlerConnector.hooks用于为ReactDomComponent组件所对应的dom节点绑定拖拽事件，必须得到调用；次参handlerMonitor用于将redux/state注入为被装饰组件的props。

5. createSourceMonitor(manager)，过滤dragDropManager.monitor可为拖拽组件DragComponent服务的方法，用于获取redux/state状态。

6. createSourceConnector(backend)=>{ hooks:{ dragSource, dragPreview }, receiveHandlerId }，生成用于为拖拽节点、拖拽预览节点绑定拖拽事件的hooks，且为hooks注入handlerId，注入为被装饰组件的props，须经开发者调用以向dom节点绑定拖拽事件。receiveHandlerId用于更新handlerId，同时各节点将重新绑定事件。

7. createSourceFactory(monitor)(spec)，将用户配置的spec钩子函数集合转化成可注册到dragDropManager.registry的DragSource。

8. registerSource(type,source,manager)，将DragSource注册到manager.registry中。

9. DropTarget(type,spec,collect,options={arePropsEqual})(DropComponent)，返回函数即投放位组件DropComponent的装饰函数，通过调用decorateHandler模块完成装饰。

10. createTargetMonitor(manager)，过滤dragDropManager.monitor可为投放位组件DropComponent服务的方法，用于获取redux/state状态。

11. createTargetConnector(backend)=>{ hooks:{ dropTarget }, receiveHandlerId }，生成用于为投放位节点绑定拖拽事件的hooks，且为hooks注入handlerId，注入为被装饰组件DropComponent的props，须经开发者调用以向dom节点绑定拖拽事件。receiveHandlerId用于更新handlerId，同时各节点将重新绑定事件。

12. createTargetFactory(monitor)(spec)，将用户配置的spec钩子函数集合转化成可注册到dragDropManager.registry的DropTarget。

13. registerTarget(type,source,manager)，将DropTarget注册到manager.registry中。

14. wrapConnectorHooks(hooks)，注入被装饰组件DragComponent, DropComponent的dragSource, dragPreview, dropTarget经wrapConnectorHooks处理后，将作用于ReactDomComponent的ref引用函数，以为其对应的dom节点绑定监听函数。

15. DragLayer(collect,options={arePropsEqual})(LayerComponent)，将redux/state拖拽状态数据通过collect(monitor)注入被装饰组件中。

### 关于设计

1. 通过ref引用获得dom节点，并为dom节点绑定事件或其他操作。

  // wrapConnectorHooks.js
  import { isValidElement } from 'react';
  import cloneWithRef from './utils/cloneWithRef';

  // 若参数element不是ReactDomComponent，报错
  function throwIfCompositeComponentElement(element) {
    // Custom components can no longer be wrapped directly in React DnD 2.0
    // so that we don't need to depend on findDOMNode() from react-dom.
    if (typeof element.type === 'string') {
      return;
    }

    const displayName = element.type.displayName ||
      element.type.name ||
      'the component';

    throw new Error(
      'Only native element nodes can now be passed to React DnD connectors.' +
      `You can either wrap ${displayName} into a <div>, or turn it into a ` +
      'drag source or a drop target itself.',
    );
  }

  function wrapHookToRecognizeElement(hook) {
    return (elementOrNode = null, options = null) => {
      // When passed a node, call the hook straight away.
      if (!isValidElement(elementOrNode)) {
        const node = elementOrNode;
        hook(node, options);
        return undefined;
      }

      // If passed a ReactElement, clone it and attach this function as a ref.
      // This helps us achieve a neat API where user doesn't even know that refs
      // are being used under the hood.
      const element = elementOrNode;
      throwIfCompositeComponentElement(element);

      // 通过ref函数引用获取dom节点，由此对dom节点绑定监听函数
      // When no options are passed, use the hook directly
      const ref = options ?
        node => hook(node, options) :
        hook;

      return cloneWithRef(element, ref);
    };
  }

  export default function wrapConnectorHooks(hooks) {
    const wrappedHooks = {};

    Object.keys(hooks).forEach((key) => {
      const hook = hooks[key];
      const wrappedHook = wrapHookToRecognizeElement(hook);
      wrappedHooks[key] = () => wrappedHook;
    });

    return wrappedHooks;
  }

  // utils/cloneWithRef.js
  import invariant from 'invariant';
  import { cloneElement } from 'react';

  // 以newRef函数引用形式拷贝ReactElement，保留之前的ref引用
  // 在react-dnd模块中，newRef函数用于为dom节点绑定监听函数
  export default function cloneWithRef(element, newRef) {
    const previousRef = element.ref;
    invariant(
      typeof previousRef !== 'string',
      'Cannot connect React DnD to an element with an existing string ref. ' +
      'Please convert it to use a callback ref instead, or wrap it into a <span> or <div>. ' +
      'Read more: https://facebook.github.io/react/docs/more-about-refs.html#the-ref-callback-attribute',
    );

    if (!previousRef) {
      // When there is no ref on the element, use the new ref directly
      return cloneElement(element, {
        ref: newRef,
      });
    }

    return cloneElement(element, {
      ref: (node) => {
        newRef(node);

        if (previousRef) {
          previousRef(node);
        }
      },
    });
  }

2. 为了不破坏原组件的设计，react-dnd通过装饰函数中调用collect为装饰组件DecoratedComponent注入state，并注入为原组件的props；且props.connectDragSource, connectDragPreview, connectDropTarget也以装饰函数的方式注入到原组件中，用于为dom节点绑定事件。

### 使用

  import React, { Component } from "react";
  import { DragDropContextProvider, DragSource, DragTarget } from "react-dnd";

  const TYPE = "TYPE";

  const customDragSource = {
    canDrag: ()=>{},
    beginDrag: ()=>{},
    isDragging: ()=>{},
    DragSource: ()=>{}
  };

  const customDropTarget = {
    canDrop: ()=>{},
    hover: ()=>{},
    drop: ()=>{}
  };

  @DragSource(TYPE, customDragSource, connect=>{
    connectDropTarget: connect.dropTarget()
  })
  @DragTarget(TYPE, customDropTarget, (connect, monitor)=>{
    connectDragSource: connect.dragSource(),
    isDragging: monitor.isDragging()
  })
  export default class CustomComponent extends Component{
    render(){
      const { connectDragSource, connectDropTarget } = this.props;
      return connectDragSource(connectDropTarget(
        <div>...</div>
      ))
    }
  };

## react-dnd-html5-backend

可装卸的拖拽事件插件。

### 主要模块

1. new HTML5Backend(manager)，通过manager注入actions, monitor, registry, context={window}。

  * 当有拖拽组件绘制到视图上时，执行setup方法，为window对象绑定handleTopDragStart, handleTopDragStartCapture, handleTopDragEnter, handleTopDragEnterCapture, handleTopDragLeaveCapture, handleTopDragOver, handleTopDragOverCapture, handleTopDrop, handleTopDropCapture, handleTopDragEndCapture监听函数。

  * 当拖拽组件调用props.connectDragSource(sourceId,node,options)为实际的拖拽节点node绑定函数时，实现上将调用HTML5Backend实例的connectDragSource方法。将拖拽节点添加到this.sourceNodes中，将拖拽节点的dragable属性置为真，为拖拽节点绑定handleDragStart, handleSelectStart，返回函数用于移除监听函数及dragable属性(react-dnd内部调用)。

  * 当拖拽组件调用props.connectDragPreview(sourceId,node,options)时，将拖拽预览节点node添加到this.sourcePreviewNodeOptions中。

  * 当拖拽组件调用props.connectDropTarget(targetId,node,options)时，将投方位节点node绑定handleDragEnter, handleDragOver, handleDrop监听函数。

  * 当节点被拖拽时：

    * 执行handleDragStart方法，将被拖拽组件的sourceId存入this.dragStartSourceIds。

    * 执行handleTopDragStart方法，执行actions.beginDrag方法，当react-dnd封装的DragComponent组件被拖拽时，通过e.dataTransfer.setDragImage(dragPreviewNode,x,y)设置预览，调用this.setCurrentDragSourceNode方法设置this.currentDragSourceNode, currentDragSourceNodeOffset, currentDragSourceNodeOffsetChanged，为window对象绑定mousemove监听函数this.endDragIfSourceWasRemovedFromDOM；当原生的文件、url或text文本被拖拽时，通过this.beginDragNativeItem方法构建NativeDragSource实例，并注册到manager.registry中，再行调用actions.beginDrag方法。

    * 执行handleTopDragStartCapture方法，清空this.currentDragSourceNode, currentDragSourceNodeOffset, currentDragSourceNodeOffsetChanged, dragStartSourceIds, 移除window对象的监听函数this.endDragIfSourceWasRemovedFromDOM。

    * 当被拖拽节点进入某投放位节点时，执行handleDragEnter方法，将投放位组件的targetId存入this.dragEnterTargetIds。

    * 执行handleTopDragEnter方法，将this.dragEnterTargetIds置为空，非火狐浏览器下执行actions.hover方法，通过e.dataTransfer.dropEffect设置降落特效。

    * 执行handleTopDragEnterCapture方法，当文件从浏览器外移到浏览器窗口中时，调用this.beginDragNativeItem方法。因为本次拖拽错过了dragStart事件。

    * 执行handleTopDragLeaveCapture方法，当文件被拖拽浏览器窗口后，又迅速移出浏览器窗口，调用this.endDragNativeItem方法，执行actions.endDrag方法，从manager.registry中移除NativeDragSource实例。

    * 当被拖拽节点悬停在某投放位节点上时，执行handleDragOver方法，将targetId存入this.dragOverTargetIds。

    * 执行handleTopDragOver方法，调用actions.hover方法，通过e.dataTransfer.dropEffect设置降落特效。

    * 执行handleTopDragOverCapture方法，清空this.dragOverTargetIds。

    * 当被拖拽节点投放在某个投放位节点上时，执行handleTopDragEndCapture方法，调用actions.endDrag。

    * 执行handleDrop方法，将targetId存入this.dropTargetIds。

    * 执行handleTopDrop方法，调用actions.hover, actions.drop，最终根据拖拽内容的不同分别调用this.endDragNativeItem或this.endDragIfSourceWasRemovedFromDOM方法。

    * 执行handleTopDropCapture方法，若拖拽的是原生的文件、url或文本时，通过调用NativeDragSource实例的mutateItemByReadingDataTransfer方法，将this.item设置为拖拽的实际内容；重置enterLeaveCounter。
  
  * 当所有可拖拽组件被移出视图时，调用HTML5Backend实例的teardown方法，为window对象解绑事件。

* EnterLeaveCounter构造函数，作为HTML5Backend实例的this.EnterLeaveCounter属性，用于在handleTopDragEnterCapture方法执行时判断是否首次移入某个投放位节点，或者在handleTopDragLeaveCapture方法执行时判断拖拽节点是否从所有可投放位节点上完全移出。

* NativeDragSource模块，生成拖拽原生file、url及文本时需要注入到manager.registry中的NativeDragSource。

* OffsetUtils, MonotonicIterpolant模块，计算预览节点的偏移量，些许费解。

### 关于设计

通过事件代理的方式由顶层window对象绑定事件处理业务，connectDragSource方法为特定拖拽节点绑定事件，完成拖拽组件Sourceid的添加和移除。


