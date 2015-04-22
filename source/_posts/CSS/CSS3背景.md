title: "CSS3背景"
date: 2015-04-21 19:55:27
categories:
tags:
---



[Background](http://www.w3.org/TR/css3-background/)是CSS Level1就开始支持的属性，用于设置元素背景相关内容。`Background`是一个复合属性，它包含如下几个子属性，

- `background-color`，规定要使用的背景颜色
- `background-position`，规定背景图像的位置
- `background-size`，规定背景图片的尺寸
- `background-repeat`，规定如何重复背景图像
- `background-clip`，规定背景的绘制区域
- `background-origin`，规定背景图片的定位区域
- `background-attachment`，规定背景图像是否固定或者随着页面的其余部分滚动
- `background-image`，规定要使用的背景图像

其中`background-size`，`background-clip`，`background-origin`，这几个属性是CSS3 Level3中关于`background`新增的属性。本篇文章将会简单阐述CSS Level1～CSS Level2中的属性，稍微详细点描述CSS Level3新增的几个属性。

# 已有属性

## `background-color`

其语法如下，

```
background-color: <color> | transparent
```

`background-color`用于设置元素的背景颜色，其值可以是一个颜色值（可以为多种颜色的表达，比如RGB、十六进制、特定值'red'等）或者为`transparent`。默认值为`transparent`，意为透明，即啥颜色也不设置。

## `background-position`

其语法如下，

```
background-position: <percentage> || <length> || [left|center|right][,top|center|bottom]
```

`background-position`用于设置背景图片的位置参数。它有多种赋值模式，可以形如`(0,0)`、`(0%,0%)`，或者是使用'left'，'top'，'center'，'right'，'bottom'等字段进行设置。

在设置`background-position`属性时，脑子里需要有这样一个概念。以左上角(0,0)的位置为坐标系的原点，向右延伸为X轴，向下延伸为Y轴。所以形如`(0,0)`、`(0%,0%)`的赋值其实是表明了背景图像距离坐标系原点的距离。而'left'意为`(0%, Y)`，'right'意为`(100%, Y)`，'center'意为`(50%, Y)`，'top'和'bottom'的含义与其相似。

## `background-repeat`

其语法如下，

```
background-repeat: repeat || repeat-x || repeat-y || no-repeat
```

`background-repeat`用于设置`background-image`（背景图片）在元素中的铺放风格的，其默认值为`repeat`，意为图片沿着右方（X轴方向）和下方（Y轴）同时平铺。

所以，`repeat-x`意为只沿着X轴方向平铺，`repeat-y`意为只沿着Y轴方向平铺，`no-repeat`意为不作任何平铺。

## `background-attachment`

其语法如下，

```
background-attachment: scroll || fixed
```

`background-attachment`用于设置背景图像是否固定活着随着页面的滚动而滚动。其默认值`scroll`，表示背景图片会随着页面滚动而滚动；取值为`fixed`时，背景图片固定不动。

## `background-image`

其语法如下，

```
background-image: none || <url>
```

`background-image`用于设置元素的背景图片，其默认值为`none`，`<url>`为背景图片的地址，可用绝对地址亦可用相对地址。


# 新增属性

## `background-size`

其语法如下，

```
background-size: auto || <length> || <percentage> || cover || contain
```

`background-size`属性用于设置背景的大小。其中，`auto`就是使用图片的原生宽高，`<length>`和`<percentage>`就是设置一个具体的值。这三个属性没什么还说的。下面说下`cover`和`contain`这两个赋值的含义。

1. `cover`：把背景图像扩展至足够大，以使背景图像完全覆盖背景区域。背景图像的某些部分也许无法显示在背景定位区域中。
2. `contain`：把图像图像扩展至最大尺寸，以使其宽度和高度完全适应内容区域。

需要注意的是，这两种取值都会导致图片失真。[这里](http://www.w3school.com.cn/tiy/c.asp?f=css_background-size&p=8)有一个示例可以体验两者的不同。

额外补充一点，当使用`<length>`或者`<percentage>`进行赋值时，我们可以只赋一个值，比如`background-size: 100px`，此时相当于`background-size: 100px 100px`。

**兼容性**：IE9+、Firefox 4+、Opera、Chrome以及Safari 5+支持`background-size`属性，所以在实际使用`background-size`属性时针对不同浏览器需要带上其前缀。

## `background-clip`

其语法如下，

```
background-clip ： border-box || padding-box || content-box
```

`background-clip`属性用于设置背景的裁剪区域，即控制元素的背景实际显示区域的大小。

1. `border-box`，此为默认值，元素背景区域包含`border`区域**及其**之内的`padding`和`content`区域。
2. `padding-box`，元素背景区域包含`padding`**及其**之内的`content`区域。
3. `content-box`，元素背景区域**仅**包含`content`区域。

各个取值的不同效果展示，可参阅这个[示例](http://www.w3school.com.cn/tiy/c.asp?f=css_background-clip)。

这里所说的`border`、`padding`、`content`等区域其实是CSS盒模型的组成部分，更多内容可参阅[CSS盒模型科普](http://gejiawen.github.io/2014/11/18/CSS/CSS%E7%9B%92%E6%A8%A1%E5%9E%8B%E7%A7%91%E6%99%AE/)。

**兼容性**：IE9+、Firefox、Opera、Chrome以及Safari支持`background-clip`属性，所以在实际使用`background-clip`属性时针对不同浏览器需要带上其前缀。

## `background-origin`

其语法如下，

```
background-origin: padding-box || border-box || content-box
```

`background-origin`属性用于设置背景的`background-position`属性相对什么来定位。

1. `padding-box`，此为默认值，相对于内边距（`padding`的外边沿）来定位。
2. `border-box`，相对于边框（`border`的外边沿）来定位。
3. `content-box`，相对于内容（`content`的外边沿）来定位。

各个取值的不同效果展示，可参阅[示例](http://www.w3school.com.cn/tiy/c.asp?f=css_background-origin)。

**兼容性**：IE9+、Firefox 4+、Opera、Chrome以及Safari 5+支持`background-origin`属性，所以在实际使用`background-origin`属性时针对不同浏览器需要带上其前缀。


## CSS3 Multiple Background

CSS3的`background`属性允许设置多个除了`background-color`之外的属性。比如，

```
background-image: url1,url2,...,urlN;
background-repeat: repeat1,repeat2,...,repeatN;
background-position: position1,position2,...,positionN;
background-size: size1,size2,...,sizeN;
backrgound-attachment: attachment1,attachment2,...,attachmentN;
background-clip: clip1,clip2,...,clipN;
background-origin: origin1,origin2,...,originN;
```

所以利于这一点我们可以仅在一个元素上同时设置多个背景来达到以往需要多个元素配合才能达到的效果。


# 参考列表

- [http://www.w3.org/TR/css3-background/](http://www.w3.org/TR/css3-background/)
- [CSS3 Background-size](http://www.w3cplus.com/content/css3-background-size)
- [CSS3 Background-clip](http://www.w3cplus.com/content/css3-background-clip)
- [CSS Background-origin](http://www.w3cplus.com/content/css-background-origin)
- [CSS3 Multiple Backgrounds](http://www.w3cplus.com/content/css3-multiple-backgrounds)



End! All rights reserved `@gejiawen`.
