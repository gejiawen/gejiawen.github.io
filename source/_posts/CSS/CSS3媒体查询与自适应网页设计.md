title: "CSS3媒体查询与自适应网页设计"
date: 2015-05-18 16:24:46
categories: [CSS]
tags: [CSS拾遗系列, css3, 响应式布局]

---

# 前序

Media Query（媒体查询）是CSS3中的新增内容，用于定义**不同媒体类型在不同CSS属性时的样式表现**。

在以前，如果我们想同一个网站对不同设备（比如PC端，手机端，平板端等等）提供支持，一般性的做法是针对不同的设备额外实现一套页面，在web端判断出访问设备类型时再路由到不同的实现。

这种做法的弊端很明显，因为额外的实现，所以后续的更新及维护都比较繁琐且成本越来越高。那么我们有没有一种方法，就是只有一份实现但是可以根据不同的设备自动做展现上的调整。Media Query为这种思路的实现提供了支持。

[这里](http://runjs.cn/detail/zybewetf)是一个例子，当改变浏览器窗口的大小时，页面上文本的颜色将会发生变化。其实现原理就是使用Media Query。

# 媒体类型（Media Type）

我们先来谈一谈**媒体类型**的相关内容。那么，何为媒体类型呢？

注意之前我们有说到，所谓**媒体查询就是针对不同媒体类型在不同CSS属性时的样式表现**。注意这句话，它有两个要素，第一是针对不同*媒体类型*，第二是针对*CSS属性*。

具体点，我们来点代码，如下，

```css
@media screen and (width: 888px) {
    p {
        color: gold;
    }
}
```

其中，

- `@media`是关键字（可以将其理解成css的一种语法糖，跟`@import`类似）
- `screen`，这个关键字就是我们所说的**媒体类型**（这里`screen`其实就是电脑屏幕）
- `width: 888px`，需要查询的CSS属性

所以上面的CSS Media Query代码要表达的意思就是：当页面在电脑屏幕上展现时，且屏幕的`width`（宽度）属性为888px时，设置所有的`p`标签元素的字体颜色为`gold`（土豪金，哈哈~）。

除了上面提到的`screen`，常见有媒体类型如下表,

| 媒体类型 | 备注 | 是否常用 |
| :--- | :--- | :--- |
| `all` | 匹配所有设备 | **是** |
| `braille` | 盲文设备 | **否** |
| `embossed` | 盲文打印 | **否** |
| `handheld` | 手持设备 | **否** |
| `print` | 打印模式 | **是** |
| `projection` | 演示模式、幻灯片等 | **否** |
| `screen` | 电脑屏幕 | **是** |
| `speech` | 演讲 | **否** |
| `tty` | 固定字母间距的网格的媒体，比如电传打字机 | **否** |
| `tv` | 电视媒体 | **否** |

看到上面关于媒体类型的表格后，其实我们常用的也就`all`、`print`、`screen`这几种类型。其中`screen`要属于最常用的媒体类型了。

在具体使用`media type`时，我们还可以使用`not`或者`only`这两个关键字修饰媒体类型，比如

```css
@media only screen and (width: 888px) {
    /* your css code */
}
```

或者

```css
@media not print and (width: 888px) {
    /* your css code */
}
```

其中，前者（`only`修饰词）表示`@media`设置的样式只对`screen`类型适用；后者（`not`修饰词）表示`@media`设置的样式对除了`print`类型之外的所有设备类型生效。


# 媒体查询（Media Query）

说完了媒体类型，我们再来说一说媒体查询。其一般的语法如下，

```css
@media screen and (width: 888px) {
    /* your css code */
}
```

媒体查询中**查询**两字的含义就体现在`screen`和`width: 888px`上（可能更加倾向后者）。

上面的示例代码中，查询页面的`width`属性，当其宽度为888px时，将应用特别设置的样式。

不过Media Query所支持的CSS属性是有限的，与一般的CSS属性并不一致，而且会有一些特别的可选项。如下表，

| 可查询属性 | 属性值类型 | 是否可用Max/Min前缀 | 描述 | 常用 |
| :---     | :---     | :---              | :--- | :---    |
| `color`  | 整数 | 是 | 定义每一组输出设备的彩色原件个数。如果不是彩色设备，则等于0 | **否** |
| `color-index` | 整数 | 是 | 定义在输出设备的彩色查询表中的条目数。如果没有使用彩色查询表，则等于0 | **否** |
| `width` | 长度 | 是 | 定义输出设备中的页面可见区域宽度 | **是** |
| `height` | 长度 | 是 | 定义输出设备中的页面可见区域高度 | **是** |
| `device-width` | 长度 | 是 | 定义输出设备的屏幕可见宽度（设备本身的宽度） | **是** |
| `device-height` | 长度 | 是 | 定义输出设备的屏幕可见高度（设备本身的高度） | **是** |
| `orientation` | `portrait`/`landscape` | 否 | 定义`height`是否大于或等于`width`。值`portrait`代表是（**竖屏**），landscape代表否（**横屏**） | **是** |
| `aspect-ratio` | 整数/整数 | 是 | 定义`width`与`height`的比率（**宽高比**） | **是** |
| `device-aspect-ratio` | 整数/整数 | 是 | 定义`device-width`与`device-height`的比率（**宽高比**）。如常见的显示器比率：4/3, 16/9, 16/10 | **是** |
| `monochrome` | 整数 | 是 | 定义在一个单色框架缓冲区中每像素包含的单色原件个数。如果不是单色设备，则等于0 | **否** |
| `scan` | `progressive`/`interlaced` | 否 | 定义电视类设备的扫描工序 | **否** |
| `grid` | 整数 | 否 | 用来查询输出设备是否使用栅格或点阵。只有`1`和`0`才是有效值，`1`代表是，`0`代表否 | **否** |
| `resolution` | dpi/dpcm | 是 | 定义设备的分辨率。如：96dpi, 300dpi, 118dpcm | **否** |

看过上面的表格之后，我们会发现其实Media Query就是为了不同设备而诞生的。就目前而言，我们常用的查询属性也就那么几个，大部分与宽高有关系（其实就是设置的展现区域大小）。

从上面的表格中，我们可以看出有部分的可查询属性还有添加`max-`或者`min-`前缀进行修饰。比如，

```css
@media screen and (min-width: 961px) and (max-width: 1200px) {
    p {
        color: pink;
    }
}
```

上面代码的含义是指，当展现页面的宽度大于960px且小于1200px时，将`p`标签的字体颜色设置为粉色。

这个例子中，我们使用了`width`和`height`这两个可查询属性，而且还使用了`max`和`min`对齐进行修饰，分别表示**最小宽度**和**最大宽度**。


# 自适应（响应式）网页设计

Responsive Web Design，国人将其翻译成响应式Web设计，个人觉得翻译成自适应Web设计可能更佳。它的意思就是**可以自动识别屏幕宽度、并做出相应调整的网页设计**。

我们可以使用CSS3的Media Query做一些自适应的网页设计。比如，

```html
<head>
    <link rel="stylesheet" media="screen and (max-width: 480px)" href="tiny.css" />
    <link rel="stylesheet" media="screen and (min-width: 481px) and (max-width: 600px)" href="small.css" />
    <link rel="stylesheet" media="screen and (min-width: 601px) and (max-width: 960px)" href="middle.css" />
    <link rel="stylesheet" media="screen and (min-width: 961px)" href="pc.css" />
</head>
```

上述代码的含义很明确，查询屏幕的宽度属性，根据不同的宽度加载不同的css文件。

有时候我们较少css文件的解析，将所有的媒体查询代码写在同一个css文件中，在html中仅作一次加载。如下，

```html
<head>
    <link rel="stylesheet" href="style.css" />
</head>
```

```css
@media screen and (max-width: 480px) {
    /* your css code */
}
@media screen and (min-width: 481px) and (max-width: 600px) {
    /* your css code */
}
/* more css code */
```

这两种做法的效果是一致的。

下面是一些示例网站，可以访问它们，然后改变浏览器的大小观察其展示的变化。

1. [Hicksdesign](http://hicksdesign.co.uk/)，不同的屏幕大小将导致页面的侧栏发生变化。
2. [A List Apart](http://www.alistapart.com/d/responsive-web-design/ex/ex-site-FINAL.html)，不同的屏幕大小将导致页面导航栏和图片行数发生变化。
3. [Colly](http://colly.com/)，不同的屏幕大小将导致页面图片的分列不同。

## 自适应布局的局限性

这里额外说一点，所谓的自适应布局远远没有这么简单，并不是靠着在不同屏幕大小上对页面布局做一些调整就可以了。

它还将面临下面的几个问题，

- 不同屏幕尺寸下，图片（视频）等资源的展示如何处理
- 在较小的屏幕尺寸下，往往需要对一些元素进行隐藏，这势必会造成流量（带宽资源）的浪费
- 即使一套页面可以自适应好几种设备了，此时一旦有更新，需要同时维护各个设备相关的css代码并且要做好协调
- ... 


# 兼容性

- ~~IE8~~（及其以下都不支持）
- IE9+
- Chrome 5+
- Opera 10+
- Firefox 3.6+
- Safari 4+

关于Media Query浏览器的兼容性，除了IE8及其以下的浏览器不支持，其他的主流浏览器基本上都支持（这里不考虑Firefox、Safari、Opera这些浏览器的低版本了）。要是实在要想在IE8上使用媒体查询，需要用到引入额外的js插件[css3-medieaqueries.js](https://github.com/livingston/css3-mediaqueries-js)。

# 参考列表

- [css3-mediaqueries](http://www.w3.org/TR/css3-mediaqueries/)
- [自适应网页设计（Responsive Web Design）](http://www.ruanyifeng.com/blog/2012/05/responsive_web_design.html)
- [什么是响应式Web设计？怎样进行？](http://beforweb.com/node/6)
- [CSS3 Media Queries 实现响应式设计](http://www.cnblogs.com/lhb25/archive/2012/12/04/css3-media-queries.html)




End! All rights reserved `@gejiawen`.
