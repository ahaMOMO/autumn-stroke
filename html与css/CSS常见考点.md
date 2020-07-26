# CSS常见考点

### 一.margin外边距重叠

> 一方用top,一方用bottom
>
> 一方用left,一方用right

#### 1.重叠场景

##### （1）相邻兄弟元素margin 合并

```html
p { margin: 10px 0; }
<p>第一行</p>
<p>第二行</p>
```

则第一行和第二行之间的间距还是 10px，因为第一行的 margin-bottom 和第二行的margin-top 合并在一起了，并非上下相加 。

##### （2）父级和第一个/最后一个子元素  

```html
.son{
	border: 1px solid red; //注意不能给.father加边框，不然就组织了margin合并
}
<div class="father">
	<div class="son" style="margin-top:80px;"></div>
</div>
<div class="father" style="margin-top:80px;">
	<div class="son"></div>
</div>
<div class="father" style="margin-top:80px;">
	<div class="son" style="margin-top:80px;"></div>
</div>
```

默认状态下，上面 3 种设置是等效的。

###### 如何阻止合并？

对于 margin-top 合并，可以进行如下操作（满足一个条件即可）：

- 父元素设置为块状格式化上下文元素；
- **父元素设置 border-top 值；**
- 父元素设置 padding-top 值；
- 父元素和第一个子元素之间添加内联元素进行分隔。  

对于 margin-bottom 合并，可以进行如下操作（满足一个条件即可）：

- 父元素设置为块状格式化上下文元素；
- 父元素设置 border-bottom 值；
- 父元素设置 padding-bottom 值；
- 父元素和最后一个子元素之间添加内联元素进行分隔；
- 父元素设置 height、 min-height 或 max-height。  

##### （3）空块级元素的 margin 合并

```html
.father { overflow: hidden; }
.son { margin: 10px 0; }
<div class="father">
	<div class="son"></div>
</div>
```

结果，此时.father 所在的这个父级<div>元素高度仅仅是 10px，因为.son 这个空<div>元素的 margin-top 和 margin-bottom 合并在一起了。  

> 由于空块级元素的 margin 合并发生不愉快事情的情况非常之少。一来，我们很少会在页面上放置没什么用的空<div>；二来，即使使用空<div>也是画画分隔线之类的，一般都是使用 border 属性，正好可以阻断 margin 合并；  

#### 2.margin合并的计算规则

我把 margin 合并的计算规则总结为“正正取大值”“正负值相加”“负负最负值” 3 句话。下面来分别举例说明。  

##### （1）正正取大值

如果是相邻兄弟合并：  

```html
.a { margin-bottom: 50px; }
.b { margin-top: 20px; }
<div class="a"></a>
<div class="b"></a>
```

此时.a 和.b 两个<div>之间的间距是 50px，取大的那个值。  

如果是父子合并：  

```html
.father { margin-top: 20px; }
.son { margin-top: 50px; }
<div class="father">
	<div class="son"></div>
</div>
```

此时.father 元素等同于设置了 margin-top:50px，取大的那个值。  

如果是自身合并：  

```html
.a {
    margin-top: 20px;
    margin-bottom: 50px;
}
<div class="a"></div>
```

则此时.a 元素的外部尺寸是 50px，取大的那个值 。

##### （2）正负值相加

如果是相邻兄弟合并：  

```html
.a { margin-bottom: 50px; }
.b { margin-top: -20px; }
<div class="a"></a>
<div class="b"></a>
```

此时.a 和.b 两个<div>之间的间距是 30px，是-20px+50px 的计算值。
如果是父子合并：  

```html
.father { margin-top: -20px; }
.son { margin-top: 50px; }
<div class="father">
	<div class="son"></div>
</div>
```

此时.father 元素等同于设置了 margin-top:30px，是-20px+50px 的计算值 。

如果是自身合并：  

```html
.a {
    margin-top: -20px;
    margin-bottom: 50px;
}
<div class="a"></div>
```

则此时.a 元素的外部尺寸是 30px，是-20px+50px 的计算值。  

##### （3）负负最负值。

如果是相邻兄弟合并：  

```html
.a { margin-bottom: -50px; }
.b { margin-top: -20px; }
<div class="a"></a>
<div class="b"></a>
```

此时.a 和.b 两个<div>之间的间距是-50px，取绝对负值最大的值。

如果是父子合并：

```html
.father { margin-top: -20px; }
.son { margin-top: -50px; }
<div class="father">
	<div class="son"></div>
</div>
```

此时.father 元素等同于设置了 margin-top:-50px，取绝对负值最大的值。  

如果是自身合并：  

```html
.a {
    margin-top: -20px;
    margin-bottom: -50px;
}
<div class="a"></div>
```

则此时.a 元素的外部尺寸是-50px，取绝对负值最大的值。  

#### 3.面试题

1.里层div距离p标签的margin值实际为多少

```html
<p>foo</p>
<div style="margin-top: 60px">       
   <div style="margin-top: 50px">bar</div>   
</div>
```

答案是：60px

### 二、CSS盒子模型

#### 1.标准盒子模型

标准盒子模型：width = content；

可通过box-sizing：content-box；设置

![image-20200722000828870](CSS%E5%B8%B8%E8%A7%81%E8%80%83%E7%82%B9/image-20200722000828870.png)

#### 2.IE盒子模型

IE盒子模型：width = content +padding +border；

可通过box-sizing：border-box；设置

![image-20200722001124067](CSS%E5%B8%B8%E8%A7%81%E8%80%83%E7%82%B9/image-20200722001124067.png)

#### 3.面试题

1.CSS的两种盒模型中，这个div的宽度是多少？

```html
<div style="width: 100px; padding:10px; margin: 15px; border: 5px solid red"></div>
```

标准盒子模型中（content-box）：width = 100px；

IE盒子模型中（border-box）：width = content + 2 * padding + 2 * border = 70px + 2 * 10px + 2 *5 = 100px;

### 三.BFC

> 传统盒模型的布局方式有三种：普通文档流布局、浮动布局和定位布局
>
> 其他布局方式：flex布局、grid布局

#### 1.概念

BFC 即 Block Formatting Contexts (块级格式化上下文)，它属于上述布局方式中的普通文档流布局。

**具有 BFC 特性的元素可以看作是隔离了的独立容器，容器里面的元素不会在布局上影响到外面的元素，并且 BFC 具有普通容器所没有的一些特性。**

#### 2.如何触发BFC

【1】根元素，即HTML元素

【2】float的值不为none

【3】overflow的值不为visible（hidden、auto、scroll）

【4】display的值为inline-block、table-cell、flex

【5】position的值为absolute或fixed 

#### 3.应用

（1）同一个BFC下外边距会发生重叠。

```html
<head>
    div{
        width: 100px;
        height: 100px;
        background: lightblue;
        margin: 100px;
    }
</head>
<body>
    <div></div>
    <div></div>
</body>
```

![image-20200725164315663](https://raw.githubusercontent.com/ahaMOMO/autumn-stroke/master/img/20200725164316.png)

从效果上看，因为两个 div 元素都处于同一个 BFC 容器下 (这里指 body 元素) 所以第一个 div 的下边距和第二个 div 的上边距发生了重叠，所以两个盒子之间距离只有 100px，而不是 200px。

首先这不是 CSS 的 bug，我们可以理解为一种规范，**如果想要避免外边距的重叠，可以将其放在不同的 BFC 容器中。**

```html
.container {
    overflow: hidden;
}
p {
    width: 100px;
    height: 100px;
    background: lightblue;
    margin: 100px;
}

<div class="container">
    <p></p>
</div>
<div class="container">
    <p></p>
</div>
//这里注意不能在上一份代码中，直接将overflow:hidden添加到div中，因为这样它们依旧位于同一个BFC（body）下
```

（2）BFC可以包含浮动元素（清除浮动）

我们都知道，浮动的元素会脱离普通文档流，来看下下面一个例子：

```html
<div style="border: 1px solid #000;">
    <div style="width: 100px;height: 100px;background: #eee;float: left;"></div>
</div>
```

![image-20200725165240952](https://raw.githubusercontent.com/ahaMOMO/autumn-stroke/master/img/20200725165242.png)

由于容器内元素浮动，脱离了文档流，所以容器只剩下 2px 的边距高度。如果使触发容器的 BFC，那么容器将会包裹着浮动元素。

```html
<div style="border: 1px solid #000;overflow: hidden">
    <div style="width: 100px;height: 100px;background: #eee;float: left;"></div>
</div>
```

效果如图：

![image-20200725165315592](https://raw.githubusercontent.com/ahaMOMO/autumn-stroke/master/img/20200725165316.png)

（3）BFC可以阻止元素被浮动元素覆盖（改进一下可以实现自适应两栏布局）

```html
<div style="height: 100px;width: 100px;float: left;background: lightblue">我是一个左浮动的元素</div>
<div style="width: 200px; height: 200px;background: #eee">我是一个没有设置浮动, 
也没有触发 BFC 元素, width: 200px; height:200px; background: #eee;</div>
```

![image-20200725165736005](https://raw.githubusercontent.com/ahaMOMO/autumn-stroke/master/img/20200725165737.png)

这时候其实第二个元素有部分被浮动元素所覆盖，(但是文本信息不会被浮动元素所覆盖) 如果想避免元素被覆盖，可触第二个元素的 BFC 特性，在第二个元素中加入 **overflow: hidden**，就会变成：

```html
<div style="height: 100px;width: 100px;float: left;background: lightblue">
    我是一个左浮动的元素
</div>
<div style="width: 200px; height: 200px;background: #eee; overflow:hidden;">
    我是一个没有设置浮动, 也没有触发 BFC 元素, width: 200px; height:200px; background: #eee;</div>
```

![image-20200725165720754](https://raw.githubusercontent.com/ahaMOMO/autumn-stroke/master/img/20200725165721.png)

这个方法可以用来实现两列自适应布局，效果不错，这时候左边的宽度固定，右边的内容自适应宽度(去掉上面右边内容的宽度)。

### 四、水平垂直居中

#### 1.定宽高

（1）absolute + 负margin（兼容好）

```css
position: absolute; 
top: 50%; 
left: 50%; 
margin-left: 宽度一半的负值; 
margin-top: 高度一半的负值;
```

（2）absolute + margin auto

```css
position: absolute; 
top: calc(50% - 高度一半); 
left: calc(50% - 宽度一半);
```

（3）absolute + calc

```css
position: absolute; 
top: calc(50% - 高度一半); 
left: calc(50% - 宽度一半);
```

#### 2.不定宽高

（1）absolute + transform

```css
position: absolute; 
top: 50%; left: 50%; 
transform: translate(-50%, -50%);
/*
(translate()方法，根据左(X轴)和顶部(Y轴)位置给定的参数，从当前元素位置移动。translate属性也可以设置百分比，其是相对于自身的宽和高)
*/
```

（2）flex

```css
display: flex;
justify-content: center;
align-items: center;
```

### 五、清除浮动的方法

先看以下例子：

```html
.father {
    border: 1px solid black;
}
.left {
    float: left;
    width: 100px;
    height: 100px;
    background-color: antiquewhite;
}
.right {
    float: right;
    width: 100px;
    height: 100px;
    background-color: aquamarine;
}
<div class="father">
    <div class="left">left son</div>
    <div class="right">right son</div>
</div>
```



![image-20200725193142996](https://raw.githubusercontent.com/ahaMOMO/autumn-stroke/master/img/20200725193144.png)

如上图中，当容器的高度为auto，且容器的内容中有浮动（float为left或right）的元素，在这种情况下，容器的高度不能自动伸长以适应内容的高度，使得内容溢出到容器外面而影响（甚至破坏）布局的现象。这个现象叫浮动溢出，为了防止这个现象的出现而进行的CSS处理，就叫CSS清除浮动。

#### （1）在浮动元素后使用一个带clear属性的空元素

```html
.clear{
	clear: both;
}

<div class="father">
    <div class="left">left son</div>
    <div class="right">right son</div>
    //这里如果后面还有标签内容，直接在浮动元素后面的标签添加clear类也是可以的。不一定要新增一个空标签
    <div class="clear"></div> 
</div>
```

#### （2）使用CSS的:after伪元素去实现

```html
.clearfix:after {
    content: "";
    display: block;
    height: 0;
    clear: both;
    visibility: hidden;
}
.clearfix {
    /* 触发 hasLayout */
    zoom: 1;
}

<div class="father clearfix">
    <div class="left">left son</div>
    <div class="right">right son</div>
</div>
```

通过CSS伪元素在容器的内部元素最后添加了一个看不见的空格"020"或点"."，并且赋予clear属性来清除浮动。需要注意的是为了IE6和IE7浏览器，要给clearfix这个class添加一条zoom:1;触发haslayout。效果如下：

![image-20200725194041306](https://raw.githubusercontent.com/ahaMOMO/autumn-stroke/master/img/20200725194043.png)

#### （3）使用CSS的overflow属性

```html
<div class="father" style="overflow: hidden; *zoom:1;">
    <div class="left">left son</div>
    <div class="right">right son</div>
</div>
```

给浮动元素的容器添加overflow:hidden;或overflow:auto;可以清除浮动，另外在 IE6 中还需要触发 hasLayout ，例如为父元素设置容器宽高或设置 zoom:1。

#### （4）给父元素添加浮动

```html
<div class="father" style="float: left;">
    <div class="left">left son</div>
    <div class="right">right son</div>
</div>
```

![image-20200725194429036](https://raw.githubusercontent.com/ahaMOMO/autumn-stroke/master/img/20200725194430.png)

给浮动元素的容器也添加上浮动属性即可清除内部浮动，但是这样会使其整体浮动，影响布局，不推荐使用。

### 六、重排与重绘

首先先了解一下浏览器的大致渲染过程：

- 构建dom树（生成dom tree）

- 根据继承和层叠规则进行元素的样式计算 （生成css rule tree）

- 构建布局树，也就是渲染树（通过dom tree + css rule tree = rendering tree），创建完渲染树之后，我们就要计算每个元素在设备视口内的确切位置和大小（这就是发生**重排**的阶段）

- 当有了布局树并且知道了每个元素的具体位置，那么就要为特定的节点生成专用的图层，生成图层,这是因为页面上会有很多的复杂效果，比如3D变换，页面滚动。

  > 满足以下条件就可以被提升为单独的一个图层：
  >
  > - 拥有层叠上下文属性的元素会被提升为单独的一层（定位fixed、opacity、css滤镜filter、z-index等）
  > - 需要剪裁（clip）的地方也会被创建为图层（）

- 之后再将各个节点绘制到屏幕上。（这就是发生**重绘**的阶段，绘制操作是由渲染引擎中的合成线程来做的，而之前的都是在GUI渲染线程中进行）

> 我们应当注意的是：**重绘不一定导致重排，但重排一定会导致重绘。**

#### 1.触发重排的操作

- 页面第一次渲染 在页面发生首次渲染的时候，所有组件都要进行首次布局，这是开销最大的一次重排

- 浏览器窗口尺寸改变

- 元素位置和尺寸发生改变的时候

- 新增和删除可见元素

- 内容发生改变（文字数量或图片大小等等）

- 元素字体大小变化

- 激活CSS伪类（例如：`:hover`）

- 设置style属性

- 查询某些属性或调用某些方法。比如说：

  offsetTop、offsetLeft、 offsetWidth、offsetHeight、scrollTop、scrollLeft、scrollWidth、scrollHeight、clientTop、clientLeft、clientWidth、clientHeight

#### 2.触发重绘的操作

- `vidibility`、`outline`、背景色等属性的改变

#### 3.如何避免或减少重排重绘？

- 样式集中改变（减少重排重绘）

  ```js
  //bad
  var left = 10;
  var top = 10;
  el.style.left = left + "px";
  el.style.top = top + "px";
  ```

  虽然现在大部分现代浏览器都会有`Flush`队列进行渲染队列优化，但是有些老版本的浏览器比如IE6这样的坑货效率依然低下： 这时我们就可以通过上面所说的利用`class`和`cssText`属性集中改变样式。

  ```js
  //good
  el.className += " className";
  ```

- 缓存布局信息（减少重排重绘）

  ```js
  // bad 强制刷新 触发两次重排
  div.style.left = div.offsetLeft + 1 + 'px';
  div.style.top = div.offsetTop + 1 + 'px';
  
  // good 缓存布局信息 相当于读写分离
  var curLeft = div.offsetLeft;
  var curTop = div.offsetTop;
  div.style.left = curLeft + 1 + 'px';
  div.style.top = curTop + 1 + 'px';
  ```

- 将dom离线化（减少重排重绘）

  一旦我们给元素设置`display:none`时，元素不会存在于渲染树中，相当于将其从页面“拿掉”，我们之后的操作将不会触发重排和重绘，这叫做DOM的离线化。

  ```js
  dom.display = 'none'
  // 修改dom样式
  dom.display = 'block'
  ```

- 将`position`属性设置为`absolute`或`fixed`（避免重排，只需重绘）

  `position`属性为`absolute`或`fixed`的元素，重排开销比较小，不用考虑它对其他元素的影响。因为它们为单独的一个图层了。

- 启用CPU加速（避免重排重绘）

  GPU 硬件加速是指应用 GPU 的图形性能对浏览器中的一些图形操作交给 GPU 来完成，因为 GPU 是专门为处理图形而设计，所以它在速度和能耗上更有效率。

  GPU 加速通常包括以下几个部分：Canvas2D，布局合成, CSS3转换（`transition`），CSS3 3D变换（`transform），WebGL和视频(`video`)。

  ```css
  /*
  * 根据上面的结论
  * 将 2d transform 换成 3d
  * 就可以强制开启 GPU 加速
  * 提高动画性能
  */
  div {
  	transform: translate3d(10px, 10px, 0);
  }
  ```

- 用opacity代替visibilty,用visibility代替display:none;

  opacity(不一定重绘)、visibility(只需重绘)、display:none(需重排重绘)