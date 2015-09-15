postid: "css-layout-width-height-responsive"
title: CSS常用布局之宽高自适应
date: 2015-03-12 21:52:27
categories: [CSS]
tags: [CSS拾遗系列, css技巧]
---

本篇文章将会介绍CSS布局中可能经常会遇到宽度或者高度自适应问题。

# 宽度自适应

我们经常会看到这样的页面，左侧（或者右侧）为固定的导航或者菜单栏，另一侧将会随着浏览器的缩放而自适应改变其大小。这种布局结构可用于顶层布局结构亦可用于某个局部功能块，常见于各种web系统（OA系统，ERP系统）等。

上述的场景即是我们所说的宽度自适应。常见的有两列布局或者三列布局（甚至是多列布局）。

这里我们用三列布局来作示例，即左右两列固定，中间一列宽度自适应。

这个其实很好实现，左侧列左浮动，右侧列右浮动，中间列不浮动即可。代码如下，

```html
<head>
<style>
    body,div {
        margin: 0;
        padding: 0;
    }
    div {
        background-color: #f00;
        height: 200px;
    }
    .left {
        float: left;
        background-color: #00f;
        width: 100px;
    }
    .right {
        float: right;
        background-color: #0f0;
        width: 100px;
    }
    .center {
        background-color: #333;
        margin: 0 100px;
        width: auto;
    }
</style>
</head>
<body>
    <div class="left">left</div>
    <div class="right">right</div>
    <div class="center">center</div>
</body>
```

效果如下，

![](http://7xkwt1.com1.z0.glb.clouddn.com/CSS常用布局之宽高自适应-001.png)

上述代码在现代浏览器中是完全没有问题的，不过在IE6中有些问题，如下，

![](http://7xkwt1.com1.z0.glb.clouddn.com/CSS常用布局之宽高自适应-002.png)

这个是只有IE6下才会有的问题，有个3px的bug，我们需要对各列的css代码进行一些兼容，

```css
.left {
    float: left;
    background-color: #00f;
    _margin-right: -3px; /*for ie6*/
    width: 100px;
}
.right {
    float: right;
    background-color: #0f0;
    _margin-left: -3px; /*for ie6*/
    width: 100px;
}
.center {
    background-color: pink;
    margin: 0 100px;
    _margin: 0 97px; /*for ie6*/
    width: auto;
}
```

这里有个在线[demo](http://runjs.cn/detail/12edciwr)。

# 高度自适应

之前有一篇文章（[CSS如何让页脚固定在页面底部](http://gejiawen.github.io/2014/12/16/CSS/CSS%E5%A6%82%E4%BD%95%E8%AE%A9%E9%A1%B5%E8%84%9A%E5%9B%BA%E5%AE%9A%E5%9C%A8%E9%A1%B5%E9%9D%A2%E5%BA%95%E9%83%A8/)）其实就是一种高度自适应的实际应用场景。

高度自适应的适用场景通常是这样的，顶栏（或者底栏）需要固定，剩余的部分能够根据浏览器的大小自适应其高度。

在现在浏览器中（包括IE7+，Chrome，Firefox等等），高度自适应可以利用绝对定位来解决。当一个元素的定位属性是`absolute`时，它将摆脱默认的文档流，其大小默认是元素内容的大小，除非手动给其设置宽高。此时我们亦可通过设置其位置属性（`top`，`right`，`bottom`，`left`）来间接改变元素的宽高。

示例代码如下，

```html
<head>
<style>
    body,div {
        margin: 0;
        padding: 0;
    }
    .top {
        background-color: #00d;
        height: 100px;
    }
    .main {
        background-color: pink;
        position: absolute;
        top: 100px;
        bottom: 0;
        left: 0;
        right: 0;
    }
</style>
</head>

<body>
    <div class="top">top</div>
    <div class="main">main</div>
</body>
```

效果如下，

![](http://7xkwt1.com1.z0.glb.clouddn.com/CSS常用布局之宽高自适应-003.png)

但是上述方案在IE6下是问题的（IE6真是翔一样的浏览器啊!!），如下图

![](http://7xkwt1.com1.z0.glb.clouddn.com/CSS常用布局之宽高自适应-004.png)

**main**块咋变成这样了啊？这是因为上述在现代浏览器中高度自适应的原理在IE6中并不适用。在IE6中，即使你将一个元素的定位属性设置成`absolute`了，此时改变其位置属性并不能改变元素的大小。

于是，在IE6中的思路就发生变化了。

在IE6中的思路是，把html和body元素的高度设定为100%，即浏览器窗口的高度，然后利用`padding-top`在html元素上挤出一点空间来，因为绝对定位的最高参照物是参照html元素的，所以可以把顶栏绝对定位在html的`padding-top`那块空间上。**在IE6中，此时页面的html元素的高度并未发生变化，但是body元素的高度减小了。**按照w3c的盒模型来解释，你给一个未明确宽高的元素设置内间距（`padding`属性），此元素将会向外膨胀撑开，使得整个元素的大小增大。但是IE6并没有按照常理出牌。

下面就是针对IE6的兼容css代码，

```css
body,div {
    margin: 0;
    padding: 0;
}
html,body {
    _height: 100%;
}
html {
    _padding-top: 100px;
}
.top {
    background-color: #00d;
    height: 100px;
    _position: absolute;
    _top: 0;
    _width: 100%;
}
.main {
    background-color: pink;
    position: absolute;
    top: 100px;
    bottom: 0;
    left: 0;
    right: 0;
    _height: 100%;
    _width: 100%;
}
```

这里有一个在线[demo](http://runjs.cn/detail/0cjweqdt)。


# 参考列表

- [CSS布局奇淫技巧之-宽度自适应](http://www.cnblogs.com/2050/archive/2012/07/30/2614852.html)
- [CSS布局奇淫技巧之-高度自适应](http://www.cnblogs.com/2050/archive/2012/07/30/2615260.html)






