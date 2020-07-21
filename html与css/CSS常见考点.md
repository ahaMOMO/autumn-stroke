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