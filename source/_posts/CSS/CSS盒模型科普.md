title: CSS盒模型科普
date: 2014-11-18 19:07:18
categories: [CSS]
tags: [CSS拾遗系列, CSS科普]
---

本文写作的目标是对css的盒模型进行科普性阐述，也是自己学习css这么长时间的关于盒模型的一个积累。当然，这篇文章还会涉及到讨论盒模型时最经常遇到的两个问题。最后将会给出一些关于css3中的`box-sizing`属性的说明和用法。

# 什么叫盒模型

那么[盒模型](http://www.w3.org/TR/CSS2/box.html)究竟是什么东西呢？

上面的链接是比较官方的介绍和解释，下面我用通俗的语言来简单说明一下。

对一个网页来说，基本上页面上所有的元素对象，其实际的呈现形式都是一个盒子形状的抽象。如下图，

![](img-01.jpg)

从图中可以看出，这个所谓的盒子其实就一个长方形（或者正方形）的抽象。从外到内，他由4层东西组成，分别是**margin（间距）**，**border（边框）**，**padding（填充）**，**content（内容）**。他们的结构特征是一层包裹着一层。

另外一点就是，有点css基础的朋友应该都知道，css盒模型是区分方向的。请记住**上右下左**这个顺序。css中一些具有四项取值的属性（比如`margin`，`padding`等）都是按照这个顺序进行赋值的。

# 盒模型的两个核心问题

每当我们谈论css盒模型的时候，往往会涉及到盒模型延伸出来的两个问题，

- W3C标准盒模型和IE盒模型
- 盒模型在几种情况下的不同表现

## 两种不兼容的盒模型

业界的前端工程师们往往都不怎么青睐IE系的浏览器，甚至对比嫌弃无比。哈，我们这里不讨论这些历史遗留原因，只讨论**W3C标准盒模型**和**IE盒模型**各自的原理。

对于盒子，我们最关心莫过于盒子的尺寸：这家伙要占多大地皮？

### W3C标准盒模型

这里仍然拿上一节中的图片来说，

![](img-01.jpg)

假设我们这个盒子的css属性如下，

```css
.box {
    width: 200px;
    height: 180;
    border: 5px solid #666
    margin: 10px
    padding: 10px;
}
```

那么，这个盒子需要占据的尺寸为，

```javascript
宽度 = margin.left + border.left + padding.left + content.width + padding.right + border.right + margin.right
    = 10 + 5 + 10 + 200 + 10 + 5 + 10
    = 300

高度 = margin.top + border.top + padding.top + content.height + padding.bottom + border.bottom + margin.bottom
    = 10 + 5 + 10 + 180 + 10 + 5 + 10
    = 280
```

而盒子的**实际大小**为，

```javascript
宽度 = border.left + padding.left + content.width + padding.right + border.right
    = 5 + 10 + 200 + 10 + 5
    = 280

高度 = border.top + padding.top + content.height + padding.bottom + border.bottom
    = 5 + 10 + 180 + 10 + 5
    = 260
```

简单的概括就是，盒子在页面占据的大小包括了`margin`，`border`，`padding`以及`content`。而盒子的**实际大小**（这里的实际大小，通过调试工具inspect可以看出来）包括`border`，`padding`以及内容区域`content`，但是不包括`margin`。

另一点需要说明的是，我们通过JavaScript代码获取某一个元素的大小时，所得到的`width`和`height`其实是盒子模型中的`content`的大小。

### IE盒模型

IE家的盒模型跟W3C标准有点不同，如下图所示

![](img-02.jpg)

从图中可以看出，IE盒模型也包含`margin`，`border`，`padding`以及`content`这几部分。（图中的`offset`指的是元素的位置偏移，后面会说到这个问题。）

IE盒模型与标准盒模型的核心差异是：**IE盒模型的`content`部分包含了`border`和`padding`**。

同样的，假设我们这个盒子的css属性如下，

```css
.box {
    width: 200px;
    height: 180;
    border: 5px solid #666
    margin: 10px
    padding: 10px;
}
```

那么，这个盒子需要占据的尺寸为，

```javascript
宽度 = margin.left + content.width + margin.right
    = 10 + 200 + 10
    = 220

高度 = margin.top + content.height + margin.bottom
    = 10 + 180 + 10
    = 200
```

而盒子的**实际大小**为，

```javascript
宽度 = content.width
    = 200

高度 = content.height
    = 180
```

> **ps**：IE家的东西非要跟别人家的东西不一样，这是为什么呢？

## 盒模型在不同情况下的表现

### 块级元素盒模型的默认宽度

如果一个块级元素未声明明确的宽度，并且是`static`或者`relative`定位，那么他的宽度将会保持100%的宽度（这里的100%指的是其最近父层元素），同时盒子的`padding`和`border`会向内推动而不是向外扩展。

但是，如果一旦明确设置了宽度为100%，那么`padding`将会向外延伸。如下图，

![](img-ex-01.png)

从图中可以看出，很明显前者没有明确设置宽度的情况下，`padding`将会向内推动，此时虽然黑底白字区域的宽度默认为100%，但是他的**实际宽度是比其父层元素小的**（没有300px）。

而后者明确设置了宽度为100%，此时`padding`将会向外延展，从而保证了元素的实际大小为100%（即300px），但是此时盒子的大小已经超过300px了。

> **ps**：这一点在平时的css编码和页面布局中经常用到，要加以留意。

### 无宽度的绝对定位盒子

未设置宽度的绝对定位的盒子的表现与之前有点不同。它的宽度只需要适合它们所包含的内容即可。因此，如果盒中只有一个单词，盒子就会像那个词的表现一样宽。如果变成两个词，盒子的宽度也会相应增加。如下图，

![](img-ex-02.png)

从图中可以看出，这种情况会持续到盒子的宽度达到父层元素宽度的100%（最近的相对定位的父元素或者浏览器窗口），如果再超出，文本就会发生折行。

对盒子来说，垂直扩展以适应包含的内容是很自然的。但是值得奇怪的是，不仅仅是不同平台下的文本表现不同，不同的浏览器处理这个问题时，也有很多怪癖。如下图，

![](img-ex-03.png)

> **ps**：从图中可以看出，相同文本折行的位置发生了变化。我个人猜测可能是因为不同浏览器对不同字体渲染的差异导致的。
> 希望知道的同学告之。

### 内联元素的盒模型

不管是块级元素还是内联元素，在页面上的展现抽象都是盒子。不过当当内联元素发生折行的时候可能表现是有一些不同。如下图，

![](img-ex-04.png)

如上所示的左`margin`把盒子推向右边，但是**只在第一行有效**，因为那是盒子的起点。`padding`应用在文本的上部或下部，但是当折行时它会忽略上面一行的`padding`并且以行高（`line-height`）要求的位置作为起点。

# box-sizing属性

## 介绍

[box-sizing](http://www.w3.org/TR/css3-ui/#box-sizing0)是[CSS3](http://www.w3.org/TR/CSS/)的Box Model（盒模型）属性之一。

`box-sizing`属性允许你以特定的方式定义匹配某个区域的特定元素。其语法如下，

```bash
box-sizing: content-box | border-box | inherit
```

例如，假如您需要并排放置两个带边框的框，可通过将`box-sizing`设置为`border-box`。这可令浏览器呈现出带有指定宽度和高度的框，并把边框和内边距放入框中。

其取值说明如下，

值 | 描述 | 说明
---|---|---
content-box | 这是由 CSS2.1 规定的宽度高度行为。宽度和高度分别应用到元素的内容框。在宽度和高度之外绘制元素的内边距和边框。 | 此值为其默认值，其让元素维持W3C的标准盒模型
border-box | 为元素设定的宽度和高度决定了元素的边框盒。就是说，为元素指定的任何内边距和边框都将在已设定的宽度和高度内进行绘制。通过从已设定的宽度和高度分别减去边框和内边距才能得到内容的宽度和高度。 | 此值让元素维持IE传统的Box Model（IE6以下版本）
inherit | 规定应从父元素继承 box-sizing 属性的值 | --

为了更能形像看出`box-sizing`中`content-box`和`border-box`两者的区别，我们来个示例图，如下所示，

![](img-ex-05.png)

`box-sizing`属性现代浏览器都支持，但IE家族只有IE8版本以上才支持，虽然现代浏览器支持`box-sizing`，但有些浏览器还是需要加上自己的前缀。

## 使用场景

这个`box-sizing`属性，我们都有哪些使用场景呢？

有人说，我们直接设置`box-sizing`的属性为`content-box`，让所有的元素都表现为标准盒模型难道不好吗？这有什么好讨论的？

哈，对于这种想法的同学，我要说，web标准的统一就靠你们啦。

上面开个玩笑，其实这个`box-sizing`属性在特定的场景下还是挺有用处的。常见的用处有如下两个，

- 特殊场景的布局
    假设我们有这样的一个场景，设置子类元素的`margin`或者`border`时，可能会撑破父层元素的尺寸，这时我就需要使用`box-sizing: border-box`来将`border`包含进元素的尺寸中，这样就不会存在撑破父层元素的情况了。
- 统一风格的表单元素
    表单中有一些input元素其实还是展现的是传统IE盒模型，带有一些默认的样式，而且在不同平台或者浏览器下的表现不一，造成了表单展现的差异。此时我们可以通过`box-sizing`属性来构建一个风格统一的表单元素。

> **ps**：关于`box-sizing`的更多实践，请参考**参考列表**中的第四项，这里就没必要重复写轮子了。


# 参考列表

- [浏览器的盒子模型 Box Model](http://blog.csdn.net/hhq163/article/details/5938228)
- [CSS 盒模型](http://css-tricks.com/the-css-box-model/)
- [Box model](http://www.w3.org/TR/CSS2/box.html)
- [CSS3 Box-sizing](http://www.w3cplus.com/content/css3-box-sizing)

End. All rights reserved `@gejiawen`.
