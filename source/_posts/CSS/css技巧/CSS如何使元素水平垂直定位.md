postid: "how-to-horizontal-and-vertical-positioning"
title: CSS如何使元素水平垂直定位
date: 2014-12-21 11:28:54
categories: [CSS]
tags: [CSS拾遗系列, css技巧]
---

作为一名前端工程师，在进行web页面开发的时候，可能会遇到这样一种问题：如何使一个元素相对父容器水平垂直居中，也就是所谓的绝对居中呢？这篇文章将会针对这个问题介绍几种常用的方法。

谈到这个问题，相信有点经验的同学们可能心里已经有了答案了。

不过，这篇文章将会从下面两种不同的应用场景分别阐述如何使用CSS让一个元素绝对居中，

- 元素有明确的高度
- 不知道元素的高度

# 元素有明确的高度

这种场景中，我们已经知道需要绝对居中的元素的高度，或者说元素的高度是确定的。其实这种场景中要使一个元素绝对居中是相对容易做到的，并且有好几种实现方法。

## 绝对定位+负margin

这是一种兼容性不错的方案，其代码如下，

```html
<div class="parent">
    <div class="centered"></div>
</div>
```

`.centered`元素就是我们需要绝对居中的元素，它的css代码如下，

```css
.centered {
    width: 100px;
    height: 100px;
    position: absolute;
    left: 50%;
    top: 50%;
    margin-top: -50px;    /* 高度的一半 */
    margin-left: -50px;    /* 宽度的一半 */
}
```

这里的技巧很容易理解，将元素绝对定位后，`left`和`top`的距离都设为50%，然后通过设置`margin-top`和`margin-left`为负值将元素移动到适当的位置，而且这里`margin-top`和`margin-left`的负值大小为元素大小为一半。

这里有一个[jsfiddle](http://jsfiddle.net/gejiawen/6hgj04bq/1/)。

显而易见，如果这里的`.centered`元素的宽高是不确定，那我们就没办法通过设置`margin-top`和`margin-left`来移动元素了。

## 只使用绝对定位

这种方案可以说是上面方案的改良版本吧，html是一样的，这里就不复述了。它的css代码如下，

```css
.centered {
    height: 100px;
    width: 200px;
    margin: auto;
    position: absolute;
    top: 0;
    right: 0;
    bottom: 0;
    left: 0;
}
```

这种方案中有两点关键点，

- 元素绝对定位，且上右下左的定位都是`0`
- 使用`margin:auto`

这里有一个[jsfiddle](http://jsfiddle.net/gejiawen/c8ypxtcg/1/)。

注意：这种方案中，css代码中的`height`和`width`其实可以不设置，但是此时`.centered`元素必须是类似`<img>`这种自身带有大小的元素。

# 不知道元素的高度

这种场景中，需要绝对居中的元素的宽高我们是不知道的，或者压根就是不确定的。此时我们就不能使用第一种场景中的两种方法了。

这种场景下，我们也有好几种实现方式。

## 绝对定位+translate

可以说这种方案是第一种场景中第一种方法的改良版本。html代码就不贴了，其css代码如下，

```css
.centered {
    position: absolute;
    left: 50%;
    top: 50%;
    transform: translate(-50%, -50%);    /* 50%为自身尺寸的一半 */
    -webkit-transform: translate(-50%, -50%);    /* 50%为自身尺寸的一半 */
    -moz-transform: translate(-50%, -50%);    /* 50%为自身尺寸的一半 */
}
```

同样，这种方法的技巧也是比较容易理解的。我们在绝对定位且`top: 50%`和`left: 50%`之后，使用css3中的`tranlaste`让元素移动自身宽高的50%，这样元素就绝对居中了，这里我们是不需要知道元素的宽高的。

这里有一个[jsfiddle](http://jsfiddle.net/gejiawen/f16nbas4/)。

由于一些客观的原因，这种看起来很美好的方法可能会遇到一些兼容性的问题，所以在实际使用中要视情况而定。

## 模拟table布局

我们可以采用模拟table布局的方式来达到使元素绝对居中的目的。其html代码如下，

```html
<div class="parent">
   <div class="centered">
       Unknown stuff to be centered.
   </div>
</div>
```

相应的css代码如下，

```css
.parent {
    display: table;
    width: 100%;
    height: 200px;
}
.centered {
    background-color: blue;
    display: table-cell;
    text-align: center;
    vertical-align: middle;
}
```

可以看出，我们让父容器采用`display: table`布局，子元素使用`display: table-cell`布局，此时，`.centered`元素其实就是一个单元格，设置`text-aligin: center`和`vertical-align: middle`后，单元格就绝对布局了。

这里有一个[jsfiddle](http://jsfiddle.net/gejiawen/f3f7xtxv/1/)。

这种在很多情况下可能会不太适用，因为table布局其实是比较恶心的，它往往会一些其他的问题。大家视情况决定吧。

## 使用`:before`

除了上面介绍的两种方法以外，我们还可以使用一种技巧性比较强的方法，其html代码和上面的是一样的，这里就不贴了。

其css代码如下，

```css
/* This parent can be any width and height */
.parent {
    text-align: center;
}

.parent:before {
    content: '';
    display: inline-block;
    height: 100%;
    vertical-align: middle;
    margin-right: -0.25em; /* Adjusts for spacing */
 }

/* The element to be centered, can also be of any width and height */
.centered {
    display: inline-block;
    vertical-align: middle;
 }
```

我们着重看一下这里`.parent:before`元素的样式，这里的`:before`伪元素起了很重要的作用，它拥有和`.parent`元素一样的高度，而且设置了`.vertical-align: middle`，同时给`.centered`元素设置了`display: inline-block`和`vertical-align: middle`。其实这里的`.parent:before`为下面的`.centered`元素提供了可以绝对居中的容器。

这里有一个[jsfiddle](http://jsfiddle.net/gejiawen/8su978hm/1/)。



# 参考列表

- [Absolute Horizontal And Vertical Centering In CSS](http://www.smashingmagazine.com/2013/08/09/absolute-horizontal-vertical-centering-css/)
- [How to Center Anything With CSS](http://designshack.net/articles/css/how-to-center-anything-with-css)
- [如何只用CSS做到完全居中](http://blog.jobbole.com/46574/)
- [Centering in the Unknown](http://css-tricks.com/centering-in-the-unknown/)
- [小tip: margin:auto实现绝对定位元素的水平垂直居中](http://www.zhangxinxu.com/wordpress/2013/11/margin-auto-absolute-%E7%BB%9D%E5%AF%B9%E5%AE%9A%E4%BD%8D-%E6%B0%B4%E5%B9%B3%E5%9E%82%E7%9B%B4%E5%B1%85%E4%B8%AD/)

