title: CSS常用布局之各种元素的水平垂直居中
date: 2015-03-13 11:22:49
categories: [CSS]
tags: [CSS拾遗系列, css技巧]
---

水平（垂直）居中是CSS布局中经常会遇到的一个场景。但是由于各个元素的差异（比如不同的元素可分为**行内元素**，**行内块级元素**，**块级元素**等），其水平或者垂直居中的策略也是不一样的。本篇文章将会详细介绍各种元素的水平垂直居中策略。

# 水平居中

针对**行内元素**及**块级元素**，各自的水平居中策略其实是不太一致的。

## 行内元素

如果被设置元素为文本、图片等**行内元素**时，可以通过给其父元素设置`text-align:center`来实现水平居中。看下面的代码，

```html
<head>
<style>
    div.txtCenter{
        text-align:center;
    }
</style>
</head>
<body>
  <div class="txtCenter">我是文本，哈哈，我想要在父容器中水平居中显示。</div>
</body>
```

## 块级元素

针对**块级元素**，比如`div`这种，我们使用`text-align:center`就不起作用了。

这里，块级元素将会分两种情况，**宽度确定**以及**宽度不确定**的块级元素，因为针对这两种情况的水平居中策略是不同的。

### 定宽块级元素

满足**定宽**条件的块级元素，我们一般可以通过设置其`margin-left`和`margin-right`属性为`auto`来实现居中。

代码如下，

```html
<head>
<style>
div{
    width: 500px;/*定宽*/
    margin: 20px auto;/* margin-left 与 margin-right 设置为 auto */
}
</style>
</head>
<body>
    <div>我是定宽块状元素，哈哈，我要水平居中显示。</div>
</body>
```

### 不定宽块级元素

在上述（**1.2.1**）的代码中，如果我们去掉明确的宽度设置，则使用`margin: 0 auto`就不能使元素水平居中了。

在实际的工作中，某些元素（块）常常由于种种原因，其内容（宽度）是不确定的，我们显然不能显式的限制其宽度。那么此时我们如何对这些元素进行水平居中呢。

通常会有三种方法，

- 使用`table`标签进行包装
- 设置`display: inline`方法
- 设置`position: relative;left: 50%`


#### 使用`table`标签进行包装

首先我们需要在要水平居中的元素的外面包装一层table，然后给table设置`margin: 0 auto`。代码如下，

```html
<head>
<style>
table {
    margin: 0 auto;
}

.div1 {
    background-color: pink;
}
.div2 {
    background-color: #ccc;
}
</style>
</head>

<div>
    <table>
        <tbody>
            <tr><td>
                <div class="div1">我被水平居中了</div>
            </td></tr>
        </tbody>
    </table>
</div>

<div class="div2">我没水平居中</div>
```

效果如下图，

![](http://7xkwt1.com1.z0.glb.clouddn.com/CSS常用布局之各种元素的水平垂直居中-001.png)

很显然这种方法有一些不足之处，**需要添加一些无意义的标签，从而造成html代码的冗余**。

#### 设置`display: inline`方法

块级元素通常包括两种，`display: block`或者`display: inline-block`。这种方法将块级元素的`display`属性设置为`inline`，然后使用`text-align: center`来实现水平居中。

示例代码如下，

```html
<head>
<style>
.container {
    text-aligin: center;
}
.div1 {
    display: inline;
}
</style>
</head>

<body>
<div class="container">
    <div class="div1">我要水平居中</div>
</div>
</body>
```

这种方法虽然不需要增加一些冗余的标签，简化了标签的嵌套深度，但是也存在一些问题。它将`display`属性设置成`inline`变成了行内元素，所以少了一些可操作性，**比如你不能给行内元素设置宽度**等。

#### 浮动加相对定位

通过给父元素设置`float`，然后给父元素设置`position:relative`和`left:50%`，子元素设置`position:relative`和`left:-50%`来实现水平居中。

示例代码如下，

```html
<style>
.container{
    float: left;
    position: relative;
    left:50%
}

.div1 {
    position: relative;
    left: -50%;
    background-color: pink;
}

.div2 {
    background-color: #ccc;
}
<body>
<div class="container">
    <div class="div1">我要水平居中</div>
</div>
<div class="div2">我没有水平居中</div>
</body>
```

这种方法可以保留块状元素仍以`display:block`的形式显示，优点不添加无语议表标签，不增加嵌套深度，但它的缺点是设置了`position:relative`，带来了一定的副作用。


# 垂直居中

元素的垂直居中不像水平居中那么复杂。这里我们只讨论简单的垂直居中问题，其他更加复杂的垂直居中问题可以参考以前的[这篇文章](http://gejiawen.github.io/2014/12/21/CSS/CSS%E5%A6%82%E4%BD%95%E4%BD%BF%E5%85%83%E7%B4%A0%E6%B0%B4%E5%B9%B3%E5%9E%82%E7%9B%B4%E5%AE%9A%E4%BD%8D/)。

## 父元素高度确定的单行文本

**父元素高度确定的单行文本**的垂直居中的方法是通过设置父元素的`height`和`line-height`属性一致来实现的。

示例代码如下，

```html
<head>
<style>
.container {
    height: 100px;
    line-height: 100px;
    background: #ddd;
}
</style>
</head>
<div class="container">我要垂直居中</div>
```

效果如下，

![](http://7xkwt1.com1.z0.glb.clouddn.com/CSS常用布局之各种元素的水平垂直居中-002.png)

## 父元素高度确定的多行文本、图片、块级元素

针对**父元素高度确定**的多行文本、图片以及块级元素，我们通常有两种间接的方法来设置其垂直居中。

### 使用table标签包装

与上面同样的原理，我们将待垂直居中的元素（块）使用`table`标签进行封装，设置`th`或者`td`标签的`vertical-align: middle`。

示例代码如下，

```html
<head>
<style>
    table td {
        height: 500px;
    }
    .wrap {
        border: 1px solid red;
        height: 500px;
        width: 650px;
    }
    .div1 {
        background-color: pink;
    }
</style>
</head>
<body>
<div class="wrap">
    <table><tbody><tr><td>
        <div class="div1">
            我要垂直居中<br>
            我要垂直居中<br>
            我要垂直居中<br>
            我要垂直居中<br>
            我要垂直居中<br>
        </div>
    </td><tr></tbody></table>
</div>
</body>
```

效果如下，

![](http://7xkwt1.com1.z0.glb.clouddn.com/CSS常用布局之各种元素的水平垂直居中-003.png)

注意一点，`td`标签默认情况下就默认设置了`vertical-align: middle`，所以我们不需要显式地设置了。

### 设置`display: table-cell`

在chrome、firefox 及IE8+的浏览器下可以设置块级元素的`display: table-cell`，激活`vertical-align: middle`属性，但注意IE6、7并不支持这个样式。

示例代码如下，

```html
<head>
<style>
.wrap{
    height: 500px;
    border: 1px solid red;
    height: 500px;
    width: 650px;
    display: table-cell; /*IE8以上及Chrome、Firefox*/
    vertical-align: middle; /*IE8以上及Chrome、Firefox*/
}
.div1 {
    background-color: pink;
}
</style>
</head>
<body>
<div class="wrap">
    <div class="div1">
        我要垂直居中<br>
        我要垂直居中<br>
        我要垂直居中<br>
        我要垂直居中<br>
        我要垂直居中<br>
    </div>
</div>
</body>
```

效果如下，

![](http://7xkwt1.com1.z0.glb.clouddn.com/CSS常用布局之各种元素的水平垂直居中-004.png)

这里，我们设置元素的`display: table-cell`属性，让其展示属性呈现单元格的特性，这样就不必再包装冗余的`table`标签代码了。这里的原理其实和第一种方法是一致的。

# 间接改变`display`属性

CSS中有一个奇怪的现象，不管元素之前的`display`属性是什么（`none`除外），当元素满足下列条件任何一个时，

- `position: absolute`
- `float: left/right`

元素会自动变为以`display:inline-block`的方式显示。此时我们可以设置元素的`width`和`height`且默认宽度不占满父元素。

我们在实际工作中可以利用这个特性，能够更加灵活的布局。

至于为什么会出现这种情况，我的猜测是，当元素设置`position: absolute`或者`float: left/right`时，元素将摆脱原始的文档流变成游离的元素，此时元素的位置可以手动任意设置，而且此时元素的大小（宽高）可以由其位置元素（`top`/`right`/`bottom`/`left`）来确定。

以上猜测若有不妥，请指正。


End. All rights reserved `@gejiawen`.


