title: "CSS3新特性概述"
date: 2015-04-13 17:03:49
categories: [CSS]
tags: [CSS拾遗系列, css3]
---

# 缘起

W3C的官方文档于2011/9/29更新了[Selectors Level 3](http://www.w3.org/TR/css3-selectors/)。时至今日，已经过去了将近4年的时间，各家浏览器厂商对CSS3的支持程度也越来越高。按理说作为一名前端从业人员不应该等某项新技术普遍被支持后才去学习，但是由于一些个人的原因（半路出家的FE？懒惰？没时间？），我还尚未对CSS3新增的新特性进行过系统的学习。

CSS作为一名web前端开发人员的必备熟练技能，实在是不应该成为自身技术栈的短板。此文便是博主学习CSS3新增特性的一个记录，不过本文将不会详细的阐述每一个新增特性，只作概述性的描述。（后面将会对每一项新增特性另作文章进行描述）

# 新增特性

本文将CSS3新增的特性按照功能分为下面的几个部分，

- 选择器的拓展
- 页面布局的加强
- 元素修饰的加强
- 开放字体的支持
- 多终端的适配
- 动画支持

# 选择器的拓展

CSS3新增了许多可使用的选择器使得前端开发人员在选择页面元素时更加灵活。新增的选择器包含如下几个方面，

- 属性选择器
- 结构伪类选择器
- UI元素状态伪类选择器
- 目标伪类选择器
- 否定选择器
- 通用兄弟选择器

具体的不打算在本文展开叙述，可参考博主之前的这篇文章[初识CSS3选择器](http://gejiawen.github.io/2015/04/09/CSS/%E5%88%9D%E8%AF%86CSS3%E9%80%89%E6%8B%A9%E5%99%A8/)。

# 页面布局的加强

## 多列布局方式

在以前，如果我们需要实现一个多列的页面布局，往往采用的方案是浮动或者绝对定位。CSS3中新增了多列布局使得页面布局更加灵活。

[这里](http://www.w3.org/TR/css3-multicol/)是多列布局的W3C官方文档。CSS3中新增了相关的属性`column-count`和`column-width`来设定具体的多列布局样式。示例代码如下，

```html
<style>
    div {
        -webkit-column-count: 2;
        -webkit-column-width: 100px;
    }
</style>

<div>
    blablablabla....
</div>
```

效果如下图，

![](1.png)

在实际的使用中，我们往往仅需要设置`column-count`或者`column-width`其中的一个属性就好，浏览器会自动根据可展示的空间对相应的元素进行分列或者设置每一列的宽度（从某种角度说，这也是一种自适应）。

除了`column-count`或者`column-width`属性之外，还有`column-gap`及`column-rule`属性可供设置，如下示例代码，

```html
<style>
    div {
        -webkit-column-count: 3;
        -webkit-column-gap: 40px;
        -webkit-column-rule: 4px solid red
    }
</style>

<div>
    blablablabla....
</div>
```

其效果如下，

![](2.png)

关于CSS3多列布局更加详细的内容，请参阅博主的这篇文章[CSS3多列布局](http://gejiawen.github.io/2015/04/14/CSS/CSS3%E5%A4%9A%E5%88%97%E5%B8%83%E5%B1%80/)。

## 可变更的盒模型

我们知道W3C的CSS2.1规范中，默认的盒模型在计算盒子的总大小的时候，元素的`border`和`padding`是被被加入到宽度和高度之中的。而IE系的老旧浏览器的行为恰恰与之相反。

CSS3规范提供了`box-sizing`属性并允许设置浏览器使用的盒模型。

简单来说`box-sizing`属性提供了`content-box`和`border-box`两种赋值来确定元素究竟使用哪一种盒模型。其中现代浏览器默认的取值是`content-box`，既符合W3C标准的盒模型。关于`box-sizing`更多的内容，请参与博主之前的一篇文章[CSS盒模型科普#box-sizing属性](http://gejiawen.github.io/2014/11/18/CSS/CSS%E7%9B%92%E6%A8%A1%E5%9E%8B%E7%A7%91%E6%99%AE/#box-sizing属性)。

## 可伸缩的布局方式

CSS3引入了可以算是一种新的盒模型：**弹性盒模型**，该模型决定一个盒子在其他盒子中的分布方式以及如何处理可用的空间。

看下面的示例代码，

```html
<style>
    .boxcontainer { 
        width: 1000px; 
        display: -webkit-box; 
        display: -moz-box; 
        -webkit-box-orient: horizontal; 
        -moz-box-orient: horizontal; 
    } 
    
    .item { 
        background: blue; 
        font-weight: bold; 
        margin: 2px; 
        padding: 20px; 
        color: #fff; 
    }
    .flex { 
        -webkit-box-flex: 1; 
        -moz-box-flex: 1; 
    }
</style>


<div class="boxcontainer">
    <div class="item">1</div>
    <div class="item">2</div>
    <div class="item">3</div>
    <div class="item flex">4</div> 
</div>
```

其展示如下，

![](3.png)


关于弹性盒模型的更多内容，请参阅博主的这篇文章[CSS3弹性盒模型]()。

# 元素修饰的加强

CSS3提供一系列属性来支持以前必须使用图片或者js操作才能实现的效果，大大的提升了页面展示效果的开发效率。

## RGBA和透明度

CSS3提供rgba来支持为元素设置透明度。如下示例代码，

```css
div {
    color: rgba(255, 0, 0, 0.75); 
    background: hsla(112, 72%, 33%, 0.68);
}
```

从示例代码可见，CSS3支持[rgb](http://baike.baidu.com/link?url=zgOBh1f7XcxGJY4sqCmoy5c3IgFZjVLlvlc1OPQNV8kAI3mADyGTOBgYAAdgdvjLnGWa4VnuSYqhSPT4hOmO-K)和[hsl](http://baike.baidu.com/link?url=DCmxScQ1REfGOBTgDrNhiIjJXfFnSAlKhjOkrQFW8e_79Ysk3pvUEKTTkunoiXeW_zSbKpGT7g-5ExUVUmH1r_)两种颜色声明方式，最后一位参数即表示透明度，其取值范围为0~1，0为完全透明1为完全不透明。

**注意**，使用`rgba`和`opacity`都能够使得某元素变得透明，但是两者有一个非常核心的区别，`opacity`属性将某元素**及其所有的子元素**都应用透明样式，而`rgba`只在被设置的元素应用透明样式并不影响其子元素。

## 圆角支持

CSS3为圆角（`border-radius`）提供了更加一般的支持，

```css
.border-radius {
    border-radius: 15px;
}
```

效果如下，

![](7.png)

关于CSS3圆角的更多内容，请参阅博主的这篇文章[CSS3圆角](http://gejiawen.github.io/2015/04/23/CSS/CSS3%E5%9C%86%E8%A7%92/)。

## 多背景图片支持

如果我们需要实现一系列背景图片堆叠在一起，一般的做法可能是需要使用多个div元素进行布局，通过调试每个div的背景样式来达到堆叠的目的。

CSS3允许你使用多个属性（`background-*`系列属性），在某一个页面元素上添加多层背景图片。（有点类似PS中图层的概念）

如下面的示例代码，

```css
div {   
    background: url(bg1.jpg) top left no-repeat,
        url(bg2.jpg) bottom left no-repeat,   
        url(bg3.jpg) center center repeat-y;   
}
```

这样就可以在同一个div元素中应用多个背景图片。具体的实践请参阅这篇文章[CSS3背景](http://gejiawen.github.io/2015/04/21/CSS/CSS3%E8%83%8C%E6%99%AF/)。

## 渐变效果支持

CSS3中提供了渐变（gradient）的支持，所谓的渐变就是两种或多种颜色之间的平滑过度。CSS3提供的渐变支持中又分为两种，**线性渐变**和**径向渐变**。

```html
<style>
    .linear-gradient {
        width: 400px;
        height: 20px;
        background-image: -webkit-gradient(linear, 0% 0%,100% 0%, from(#2A8BBE),to(#FE280E));
    }
    .radial-gradient {
        width: 100px;
        height: 100px;
        background: -webkit-gradient(radial, 50 50, 50, 50 50, 0, from(blue), to(red));
    }
</style>
<div class="linear-gradient"></div>
<div class="radial-gradient"></div>
```

其效果如下下图，

![](4.png)

关于CSS3渐变的更多内容，请参阅博主的这篇文章[CSS3渐变]()。

## 阴影和反射效果

我们可以使用`text-shadow`和`box-shadow`来达到阴影的目的。

如下示例代码，

```css
 .class1{ 
    text-shadow:5px 2px 6px rgba(64, 64, 64, 0.5);
 } 

 .class2{ 
    box-shadow:3px 3px 3px rgba(0, 64, 128, 0.3);
 }
```

其中`text-shadow`是针对文字进行阴影设置，而`box-shadow`是针对容器元素进行阴影设置。同时`text-shadow`和`box-shadow`都支持同时对一个元素设置多个阴影属性（即所谓的多重阴影）。例如，

```css
.class1{ 
    text-shadow:1px 1px 2px #c10ccc,
        1px 1px 2px #648cb4,
        1px 1px 6px #cc150c;
 } 
```

效果如下，

![](5.png)

再来谈谈反射效果，所谓的反射其实看起来就跟水中的倒影一样，如下图的效果，

![](6.png)

其实它的设置很简单，

```css
.classReflect{ 
    -webkit-box-reflect: below 10px -webkit-gradient(linear, left top, left bottom, from(transparent), to(rgba(255, 255, 255, 0.51))); 
}
```

关于CSS3中阴影和反射更多的内容，请参阅博主的另一篇文章[CSS3阴影和反射]()。

# 开放字体的支持

CSS3提供`@font-face`特性为页面自定义字体的展示提供支持。通过`@font-face`你可以在页面中嵌入不同的自定义字体，为不同的元素应用不同的字体。

其一般用法如下，

```css
/* 自定义字体 */
@font-face { 
    font-family: BorderWeb; 
    src:url(BORDERW0.eot); 
}
@font-face { 
    font-family: Runic; 
    src:url(RUNICMT0.eot); 
}

/* 为不同的元素应用不同的字体 */
.border {
    font-family: "BorderWeb";
    font-size: 35px; color: black;
} 
.event {
    font-family: "Runic";
    font-size: 110px;
    color: black;
}
```

关于`@font-face`更多的内容请参阅博主之前的另一篇文章[WebFont与页面ICON图标研究](http://gejiawen.github.io/2015/03/04/CSS/WebFont%E4%B8%8E%E9%A1%B5%E9%9D%A2ICON%E5%9B%BE%E6%A0%87%E7%A0%94%E7%A9%B6/)。


# 多终端的适配

之前为了适配不同的设备（主要是不同设备的屏幕不一样），可能需要为不同设备准备不同的css文件。CSS3提供了更加方便的方式，通过媒体查询（media queries）可以让你为不同的设备基于它们的能力定义不同的样式。

比如，在可视区域小于480像素的时候，你可能想让网站的侧栏显示在主内容的下边，这样它就不应该浮动并显示在右侧了，

```css
#sidebar {   
    float: right;   
    display: inline; /* IE Double-Margin Bugfix */  
}   
  
@media all and (max-width:480px) {   
    #sidebar {   
        float: none;   
        clear: both;   
    }   
}
```

关于CSS3媒体查询的更多内容，请参阅博主的这篇文章[CSS3媒体查询]()。


# 动画支持

web开发中要在页面上实现动画有许多种途径。CSS3为我们提供了一系列方便的方法。在CSS3中，有如下三个关键字可以用来实现动画，

- `transition`，意为**过渡**
- `transform`，意为**变换**
- `animation`，意为**动画**

这三者之间稍有区别，关于他们更加详细的用法，请参阅博主的这篇文章[CSS动画与@keyframes研究]()。


End! All rights reserved `@gejiawen`.
