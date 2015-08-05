title: CSS的长度单位参考
date: 2015-03-03 11:38:36
categories: [CSS]
tags: [CSS拾遗系列, css科普]
---

CSS中有相当一部分属性的值是根据长度来定义的，而长度单位在CSS中并不是统一的，本文将详细介绍CSS（及CSS3新增的）几种长度单位，并给出常用场景。

按照大的类别来分，CSS的长度单位可分为绝对长度单位和相对长度单位。

# 绝对长度单位

绝对长度单位是一个固定的值，它反应一个真实的物理尺寸。那么，常见的绝对长度单位有哪些呢？

- `in`，英寸
- `cm`，厘米
- `mm`，毫米
- `pt`，points
- `pc`，Picas

这些单位都是拥有真实的物理尺寸的以及确定的换算关系，比如，`1in = 2.54cm`。我们平时在书写css代码时，当然也可以直接使用这些单位，并且也能够在屏幕上表现。不过由于这些单位都是绝对长度单位，往往不利于页面屏幕的渲染，这些绝对长度单位更多的使用场景往往是被用在印刷、打印等方向。


# 相对长度单位

CSS相对长度单位中的相对二字，表明了其长度单位会随着它的参考值的变化而变化，不是固定的。下面是常用的一些相对长度单位，

- `px`，Pixels像素
- `em`，元素的字体高度
- `ex`，x-height，字母x的高度
- `%`，百分比

## px

[参考文章1](http://www.w3cplus.com/css/the-lengths-of-css.html)中将`px`归为绝对长度单位，我并不太认同这一点。

因为**像素**单位其实是和设备屏幕的分辨率是直接相关的。比如，有屏幕上有一个div，其宽度为100px，其在分辨率为800x600的屏幕上占据的宽度为1/8，而在1000x800的屏幕上占据的宽度是1/10。

在web开发中，像素现在仍然是典型（主流的）度量单位。当然，你可以在web开发的过程中采用其他的单位，但是往往这些单位在渲染时都被映射换算成像素。


## em

单位`em`的含义最初是指基于当前字体大写字母"M"的尺寸。所以当改变font-size的大小时，这个长度单位将会发生变化。

现代所有的浏览器中，都会有这样的一个默认值，即`1em = 16px`，所以即使你忘记了设置`font-size`也不要紧。

关于em单位有下面几点需要注意，

- 基于当前元素的(如果没设置就是继承其父元素的)font-size
- em单位具有级联关系


比如，

```html
<div style="font-size: 12px;">
    <span style="font-size: 2em">em单位</span>
</div>
```

如上代码中，`span`标签中设置了`font-size: 2em`，因为其父标签设置了`font-size: 12px`，所以`span`标签的结果就是`font-size: 24px`。

再比如，

```html
<div id="div1" style="font-size: 12px">
    <div id="div2" style="font-size: 1.2em">
        <div id="div3" style="font-size: 1.2em"></div>
    </div>
</div>
```

inspect结果如下，

![](http://7xkwt1.com1.z0.glb.clouddn.com/CSS的长度单位参考-001.png)
![](http://7xkwt1.com1.z0.glb.clouddn.com/CSS的长度单位参考-002.png)

可见div3和div2的font-size并不一致。其原因就是因为em长度单位有级联效果。上述代码的计算规则如下，

- div1：font-size，12px
- div2：font-size，12 * 1.2 = 14.4px
- div3：font-size，12 * 1.2 * 1.2 = 17.28px

至于为什么图中会带有那么多小数点，那是因为运算浮点数都是不精确的，或者说浮点数的运算都是有一定的精确度的。

接上，所以div3中的font-size：1.2em，此时这个em是相对div2的，而div2的font-size是相对div1的。div3中的em已经不是div2中的em了。

## 百分比

**%（百分比）**对于`font-size`、`line-height`等属性是基于其父元素的`font-size`的，而对于`text-indent`、`margin`、`padding`、`width`等属性则是基于父元素的宽度的。

比如，

```css
.box {
    line-height: 130%;
    font-size: 130%;
}
```

这里的`.box`表示当前元素的行高和字体大小是其继承的（一般是当前元素的最近父元素）`font-size`的130%，其实就等同于1.3em。


# CSS3中新增的长度单位

CSS3中新增的长度单位有如下，

- `ch`，字符0的宽度
- `rem`，根元素（html）的font-size
- `vw`，viewpoint width，视窗宽度，1vw=视窗宽度的1%
- `vh`，viewpoint height，视窗高度，1vh=视窗高度的1%
- `vmin`，`vmax`等

新增的viewpoint相关的单位一般是针对移动设备的。本文不打算针对其作详细描述，欲知详情请参见[这篇文章](https://css-tricks.com/viewport-sized-typography/)。

`rem`其实跟`em`是一回事，只不过`rem`多了一个限制条件，它表示根元素（html元素）的font-size。

`rem`和`em`的区别，我们可以通过下面这个例子来看一看，

```html
<html style="font-size: 12px;">
<body>
    <div id="div1" style="font-size: 16px">
        <div id="div2" style="font-size: 1.2em"></div>
        <div id="div3" style="font-size: 1.2rem">
            <div id="div4" style="font-size: 1.2em"></div>
        </div>
    </div>
</body>
</html>
```

此时，

- div2的font-size: `16px*1.2em=19.2px`
- div3的font-size: `12px*1.2rem=14.4px`
- div4的font-size: `12px*1.2rem*1.2em=17.28px`

可见，`rem`不存在级联关系，而`em`存在级联关系。

值得注意的是，IE8及以下、Safari 4、IOS 3.2等不支持`rem`单位。


# 参考列表

- [CSS的长度单位](http://www.w3cplus.com/css/the-lengths-of-css.html)
- [理解css中的长度单位](http://www.qianduan.net/understand-the-unit-of-length-in-the-css.html)
- [CSS长度单位参考](http://blog.sina.com.cn/s/blog_4b7463e00100b33g.html)

End. All rights reserved `@gejiawen`.


