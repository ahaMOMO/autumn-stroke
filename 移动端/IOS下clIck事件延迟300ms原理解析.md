# IOS下clIck事件延迟300ms原理解析

### 1.原因

**由于区分单击事件和双击屏幕缩放的原因造成的**

2007年苹果发布首款iphone上IOS系统搭载的safari为了将适用于PC端上大屏幕的网页能比较好的展示在手机端上，使用了双击缩放(double tap to zoom)的方案，比如你在手机上用浏览器打开一个PC上的网页，你可能在看到页面内容虽然可以撑满整个屏幕，但是字体、图片都很小看不清，此时可以快速双击屏幕上的某一部分，你就能看清该部分放大后的内容，再次双击后能回到原始状态。

双击缩放是指用手指在屏幕上快速点击两次，iOS 自带的 Safari 浏览器会将网页缩放至原始比例。

原因就出在浏览器需要如何判断快速点击上，当用户在屏幕上单击某一个元素时候，例如跳转链接，此处浏览器会先捕获该次单击，但浏览器不能决定用户是单纯要点击链接还是要双击该部分区域进行缩放操作，所以，捕获第一次单击后，浏览器会先Hold一段时间t，如果在t时间区间里用户未进行下一次点击，则浏览器会做单击跳转链接的处理，如果t时间里用户进行了第二次单击操作，则浏览器会禁止跳转，转而进行对该部分区域页面的缩放操作。那么这个时间区间t有多少呢？在IOS safari下，大概为300毫秒。这就是延迟的由来。造成的后果用户纯粹单击页面，页面需要过一段时间才响应，给用户慢体验感觉，对于web开发者来说是，页面js捕获click事件的回调函数处理，需要300ms后才生效，也就间接导致影响其他业务逻辑的处理。

### 2.浏览器开发商的解决方案

##### 2.1  禁止缩放

当HTML文档头部包含如下`meta`标签时：

```html
<meta name="viewport" content="user-scalable=no">
<meta name="viewport" content="initial-scale=1,maximum-scale=1">
```

表明这个页面是不可缩放的，那双击缩放的功能就没有意义了，此时浏览器可以禁用默认的双击缩放行为并且去掉300ms的点击延迟。

缺点：

就是必须通过**完全禁用缩放来达到去掉点击延迟的目的**，然而完全禁用缩放并不是我们的初衷，我们只是想禁掉默认的双击缩放行为，这样就不用等待300ms来判断当前操作是否是双击。但是通常情况下，我们还是希望页面能通过双指缩放来进行缩放操作，比如放大一张图片，放大一段很小的文字。

##### 2.2  更改默认的视口宽度

一开始，为了让桌面站点能在移动端浏览器正常显示，移动端浏览器默认的视口宽度并不等于设备浏览器视窗宽度，而是要比设备浏览器视窗宽度大，通常是980px。我们可以通过以下标签来设置视口宽度为理想宽度。

```js
<meta name="viewport" content="width=device-width">
```

因为双击缩放主要是用来改善桌面站点在移动端浏览体验的，而随着响应式设计的普及，很多站点都已经对移动端坐过适配和优化了，这个时候就不需要双击缩放了，如果能够识别出一个网站是响应式的网站，那么移动端浏览器就可以自动禁掉默认的双击缩放行为并且去掉300ms的点击延迟。如果设置了上述`meta`标签，那浏览器就可以认为该网站已经对移动端做过了适配和优化，就无需双击缩放操作了。

这个方案相比方案一的好处在于，它没有完全禁用缩放，而**只是禁用了浏览器默认的双击缩放行为，但用户仍然可以通过双指缩放操作来缩放页面。**

##### 2.3  CSS touch-action

`touch-action`这个CSS属性。这个属性指定了相应元素上能够触发的用户代理（也就是浏览器）的默认行为。如果将该属性值设置为`touch-action: none`，那么表示在该元素上的操作不会触发用户代理的任何默认行为，就无需进行300ms的延迟判断。

### 3.现有的解决方案

##### 3.1 指针事件的polyfill

现在除了IE，其他大部分浏览器都还不支持指针事件。有一些JS库，可以让我们提前使用指针事件，比如

- Google 的 [Polymer](https://link.jianshu.com?t=https://github.com/Polymer/PointerEvents)
- 微软的 [HandJS](https://link.jianshu.com?t=http://handjs.codeplex.com/)
- [@Rich-Harris](https://link.jianshu.com?t=https://github.com/Rich-Harris) 的 [Points](https://link.jianshu.com?t=https://github.com/Rich-Harris/Points)

然而，我们现在关心的不是指针事件，而是与300ms延迟相关的CSS属性`touch-action`。由于除了IE之外的大部分浏览器都不支持这个新的CSS属性，所以这些指针事件的polyfill必须通过某种方式去模拟支持这个属性。一种方案是JS去请求解析所有的样式表，另一种方案是将`touch-action`作为html标签的属性。

##### 3.2 FastClick

[FastClick](https://link.jianshu.com?t=https://github.com/ftlabs/fastclick) 是 [FT Labs](https://link.jianshu.com?t=http://labs.ft.com/) 专门为解决移动端浏览器 300 毫秒点击延迟问题所开发的一个轻量级的库。FastClick的实现原理是在检测到touchend事件的时候，会通过DOM自定义事件立即出发模拟一个click事件，并把浏览器在300ms之后的click事件阻止掉。

### 4.FasstClick原理

首先了解一下移动端触摸响应事件的顺序为：touchstart->touchmove->touchend->mousemove->mousedown->mouseup->click。

fastclick在touched阶段调用event.preventDefault阻止浏览器默认行为（这里的默认行为为原来的click事件），然后通过document.dispatchEvent创建一个mouseEvents,然后通过eventTarget.dispatchEvent触发对应目标元素上绑定的click事件。（简单来说就是在检测到tochend事件时，阻止原来的click事件，然后通过自定义dom事件立即发出模拟一个click事件去实现）。

但是，touched阶段之后不能每次都触发事件，因为有可能用户在上下滑动，而不是点击。

所以fastclick使用时间偏差去判断。它分别记录touchstart和touched的时间戳，如果它们的时间差大于700ms，则认为是滑动操作，否则是点击操作。

### 5.点击穿透





