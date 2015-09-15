postid: "css3-shadow-and-reflect"
title: "CSS3阴影和反射"
date: 2015-05-05 10:28:45
categories: [CSS]
tags: [CSS拾遗系列, css3]

---

对网页设计人员来说，阴影可能是比较常用的一种用来增强页面修饰的手段（可能反射用的频率较少）。在CSS3支持阴影和反射效果之前，大部分的解决方案都是通过图片来展现所需要的效果，但是我们知道使用图片往往会有各种弊端。本篇文章将会阐述CSS3中阴影和反射的一系列使用方法。

# CSS3阴影

在CSS3中，与阴影相关的具体属性有`text-shadow`和`box-shadow`，分别表示文本阴影和容器阴影。

## 文本阴影

实际上，`text-shadow`并不是在CSS3中新增的的新属性，在CSS2.0时就已经有了这个属性了，之后在CSS2.1不知何故被移除了，最终在CSS3中又重新收纳了这个属性。

语法，

```
text-shadow：none | <shadow> [ , <shadow> ]*
```

其中，`<shadow> = <length>{2,3} && <color>?`，即`<shadow>`可以设置为2个（或者3个）的长度单位（只要是CSS中允许的长度标识单位皆可，比如`px`，`em`等）加上一个颜色值。其默认值为`none`。下面是一个使用`text-shadow`的例子，

```css
text-shadow: 1px 2px 2px #5678AF
```

其效果如下图，

![](http://7xkwt1.com1.z0.glb.clouddn.com/CSS3阴影和反射-001.png)

参数说明如下，

- 默认值`none`，表示没有阴影。
- 第一个长度值用于设置文本阴影的**水平偏移值**，可以为负值。
- 第二个长度值用于设置文本阴影的**垂直偏移值**，可以为负值。
- 第三个长度值用于设置阴影的**模糊半径**，**不**可为负值。此参数可省略。
- 颜色值用于标识阴影的具体颜色。

## 容器阴影

`box-shadow`用于给容器设置阴影。其用法比`text-shadow`相比要复杂一点。其具体语法如下，

```
box-shadow：none | <shadow> [ , <shadow> ]*
```

其中，`<shadow> = inset? && [ <length>{2,4} && <color>? ]`，即`<shadow>`可设置：是否为内阴影，2个~4个的长度单位加上一个颜色值。下面是一个使用`box-shadow`的示例，

```css
box-shadow: 2px 2px 5px 1px rgba(0,0,0,.6);
```

其效果如下图，

![](http://7xkwt1.com1.z0.glb.clouddn.com/CSS3阴影和反射-002.png)

参数说明如下，

- 默认值`none`，表示没有阴影。
- 第1个长度值用来设置对象的阴影水平偏移值。可以为负值。
- 第2个长度值用来设置对象的阴影垂直偏移值。可以为负值。
- 如果提供了第3个长度值则用来设置对象的阴影模糊值。不允许负值。
- 如果提供了第4个长度值则用来设置对象的阴影外延值。
- `inset`用于设置对象的阴影类型为内阴影。该值为空时，则对象的阴影类型为外阴影 。

这里对`box-shadow`的第四个长度参数多叨唠几句。第四个长度参数用于设置阴影的外延值。这个参数经常被忽略，其实在有些场景下还是挺有用处的。此参数的默认值为0，我们将此值赋值为阴影模糊半径的**负值**来抵消模糊值，从而得到单边阴影的效果。下面是一个例子，

```css
.div1 {
    box-shadow: 0 8px 6px #565656;
}
.div2 {
    box-shadow: 0 8px 6px -6px #565656;
}
```

其效果如下，

![](http://7xkwt1.com1.z0.glb.clouddn.com/CSS3阴影和反射-003.png)

左侧的div1未设置阴影外延值，右侧的div2设置了一个负的外延值，且数值与阴影模糊半径一致。


## 多重阴影

无论是`text-shadow`还是`box-shadow`，都是支持多重阴影的。就是可以给`text-shadow`或者`box-shadow`属性设置多组阴影值。如下，

```css
.text-shadow {
    text-shadow: 1px 1px 5px gold, 5px 5px 2px pink, -2px -2px 6px green;
}
.box-shadow {
    box-shadow: 2px 4px 6px pink, -2px -4px 6px green;
}
```

其效果如下，

![](http://7xkwt1.com1.z0.glb.clouddn.com/CSS3阴影和反射-004.png)

从效果图中可以看出，我们可以利用多重阴影搭配不同的阴影颜色做出各种玄幻的效果。唯一限制你的就是你的想象力。😂

## 兼容性

描述阴影的两个CSS3属性在现代浏览器上表现良好（需要带上不同浏览器内核的私有前缀），但是老旧的浏览器并不支持。

文本阴影`text-shadow`的兼容性如下图，

![](http://7xkwt1.com1.z0.glb.clouddn.com/CSS3阴影和反射-005.png)

需要注意的是，文本阴影IE系浏览器到IE10才支持，之前版本的IE浏览器中要实现阴影效果需要使用IE的相关私有实现（即IE滤镜的用法，不过这里不准备对其进行描述，想要了解更多关于IE滤镜的内容请自行查阅相关资料）。

容器阴影`box-shadow`的兼容性如下图，

![](http://7xkwt1.com1.z0.glb.clouddn.com/CSS3阴影和反射-006.png)


# CSS3反射

CSS3中对反射（或者说倒影）提供了支持。那么，什么是反射（或者说倒影）？让我们先来看张效果图。

![](http://7xkwt1.com1.z0.glb.clouddn.com/CSS3阴影和反射-007.png)

CSS3中提供此效果的属性叫做`box-reflect`，其语法如下，

```
box-reflect：none | <direction> <offset>? <mask-box-image>?
```

- `none`，为默认值，表示无反射。
- `<direction>`，反射的方向，可选值为`above`，`below`，`left`，`right`。
- `<offset>`，表示本体和反射之间的间隔，可设置为长度值也可设置为百分比。此参数可省略，默认为0。
- `<mask-box-image>`，用于设置反射的遮罩，可设置为渐变或者一个图片。此参数可省略。

上述效果对应的css代码如下，

```css
.box-reflect {
    -webkit-box-reflect: below 20px -webkit-linear-gradient(top, transparent, rgba(255,255,255,.3));
    width: 560px;
    font-size: 4em;
    color: gold;
}
```

在应用反射时，有一点需要注意一下，就是我们要为反射的方向（即`<direction>`参数的方向）预留足够的空间，否则会导致页面看起来无反应，因为没有足够的空间用来展示反射。

其浏览器兼容性如下图，

![](http://7xkwt1.com1.z0.glb.clouddn.com/CSS3阴影和反射-008.png)

CSS3反射可以很简单的解决之前必须使用图片才能解决的倒影问题，不过由于一些浏览器的支持问题，使得这个属性貌似使用的不是很广泛。

这里有一篇w3cplus上介绍反射的文章[使用CSS3制作倒影](http://www.w3cplus.com/css3/css3-box-reflect.html)，我觉得挺基础的，有兴趣的可以参阅。

*文章中所有的浏览器兼容性图片资源均来自[http://www.w3chtml.com](http://www.w3chtml.com/)。*




