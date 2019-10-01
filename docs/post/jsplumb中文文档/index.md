
> 前言

以下jsPlumb文档介绍基于jsPlumb2.1.7版本，这是一个开源的流程图或拓扑图绘制工具库，目前网上可用的中文翻译文档并不齐全，所以开着翻译工具给强行翻译一下。
#### jsPlumb
- jsPlumb使用SVG为开发者提供了一个可以形象化连接页面元素的工具
- jsPlumb库不需要外部依赖
- jsPlumb最后一个兼容IE8的版本为1.7.x，此后版本将只支持在现在浏览器中使用，但官方会继续维护1.7.x版本

#### 文件引入和基本设置
**浏览器兼容性**

- 在IE9中使用jQuery 1.6.x 和1.7.x版本会导致SVG实现的时候hover事件出现问题
- jquery2.0不再支持IE8及以下
- safari5.1在禁止鼠标事件的时候会有一个SVG的BUG

**文件引入**

老版本的需要引入jquery和jquery-ui两个作为依赖库
```js
<script src="jsPlumb-2.1.7-min.js" type="text/javascript"></script>
```
**初始化**

jsplumb在DOM渲染完毕之后才会执行，所以需要为jsplumb的执行代码绑定一个ready事件：
```js
jsPlumb.ready(function() {
    ...         
    // your jsPlumb related init code goes here
    ...
});
```
jsplumb默认是注册在浏览器窗口的，将整个页面提供给我们作为一个实例，但我们也可以使用getInstance方法建立页面中一个独立的实例：
```js
var firstInstance = jsPlumb.getInstance();
```
建立实例之后我们可以为其设置一些默认值：
```js
firstInstance.importDefaults({
  Connector : [ "Bezier", { curviness: 150 } ],//连接器类型
  Anchors : [ "TopCenter", "BottomCenter" ]//锚点位置
});
 
firstInstance.connect({
  source:"element1",//源端点 
  target:"element2",//目标端点
  scope:"someScope" //可拖拽范围
});
```
其实在使用getInstance的时候还可以随意设置默认的参数：
```js
var secondInstance = jsPlumb.getInstance({
  PaintStyle:{ //端点参数
    lineWidth:6, 
    strokeStyle:"#567567", 
    outlineColor:"black", 
    outlineWidth:1 
  },
  Connector:[ "Bezier", { curviness: 30 } ],
  Endpoint:[ "Dot", { radius:5 } ],
  EndpointStyle : { fillStyle: "#567567"  },//端点参数
  Anchor : [ 0.5, 0.5, 1, 1 ]//锚点
});
 
secondInstance.connect({ 
  source:"element4", 
  target:"element3", 
  scope:"someScope"   
});
```
**元素ID**

jsplumb需要使用元素的ID属性来实现交互，如果用户没有设置ID，它将自动帮用户设置一个ID，所以还是建议用户自己为元素设置一个合适的ID。

因为jsplumb的这个特性，所以你必须告知jsplumb某一元素的ID是否被改变：

- jsPlumb.setId(el, newId) 使用这个方法设置ID会让jsplumb自动跟随ID的变化
- jsPlumb.setIdChanged(oldId, newId) 如果元素的ID已经被改变，可以使用此方法来更新ID

#### 基本概念
**锚(Anchor)**：一个位置，放置端点的地方，相对于一个元素的来源，您不需要自己硬编码来创建它，jsPlumb提供给您各种功能，您只需要按照您的需要创建它就可以了。它没有可视化的显示，只是一个逻辑位置，可以使用锚的id来引用它，jsPlumb支持这样做，并且您可以使用坐标来表示[x,y,x方向,y方向]

**端点(Endpoint )**：链接的一端的可视化表示，您可以创建并可以链接他们；您可以让他们支持拖拽，或者您可以直接使用jsPlumb.connect()在创建链接时直接创建它们。

**连接器(Connector)**：链接两个元素的线，页面的可视化表示，jsPlumb有三种默认类型:Bezier曲线，直线，和流程图链接器，您不用去处理连接器，当您需要使用它们时，您只需要定义它们即可。

**覆盖物(Overlay)**：一个UI组件，是用来是用来装饰连接器，例如标签、箭头等。

**1. 锚(Anchors)**

锚的概念是指：定义一个链接线能够链接的点，jsPlumb有9个默认的锚点位置，您可以使用它们去链接元素，包括每个方向并排两个位置加中间位置，这些位置在jsplumb底层代码中都是以阵列语法表示的，[X,Y,DX,DY],其中X,Y是在区间[0,1]指定锚的位置，DX和DY是在区间[-1,1]指定曲线的事件锚的方向坐标，例如[0，0.5，-1，0]定义了一个“LeftCenter”连接器的曲线，从锚点单向向左的曲线.同样，[0.5，0，0，-1]，定义一个“CenterTop”锚连接器所产生的向上的曲线。

**2. 动态锚(Dynamic Anchors)**

这些都是可以在若干地点之一定位的锚点，当你每次移动一个元素时，会自动选择一个最合适的位置，没有特殊的语法来创建一个DynamicAnchor，你只需要提供一个独立的锚位置，例如数组，

jsPlumb提供一个动态的锚，定名为“AutoDefault”默认位置有：TopCenter，RightMiddle，BottomCenter和LeftMiddle