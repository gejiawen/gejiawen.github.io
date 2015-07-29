title: CSS如何让页脚固定在页面底部
date: 2014-12-16 20:35:36
categories: [CSS]
tags: [CSS拾遗系列, css技巧]
---

在进行Web开发实现页面时，可能会遇到这样一种场景：将页脚footer固定在页面的底部，如果页面的主体不能充满屏幕高度，则footer位于**屏幕的底部**；如果页面的主体超出了屏幕高度，则footer始终位于**页面底部**。

场景的示意如下图，

![](img-01.png)

那么如何将页脚始终固定在页面底部呢？

一般来说，有两类方法可以达到此目的，一种是仅仅使用css技巧并配合特定的html结构；另一种是使用js代码操作dom元素。

本文将介绍三种使用css技巧的方法来达到此目的。

# 第一种方法

第一种方法的原理是，

> 页面中的`html`，`body`，`container`都必须满足`height:100%`，这样就可以占满整个屏幕（页面），`footer`使用相对定位`bottom:0`，固定在页面底部，页面主体`page`容器容易必须要设置一个大于等于`footer`高度的`padding-bottom`，目的就为了将`footer`的高度计算在`page`容器中，这样一来`footer`就会始终固定在页面底部了。

我们先来看下html结构，

```html
<div id="container">
    <div id="header">Header Section</div>
    <div id="page" class="clearfix">
        <div id="left">Left Sidebar</div>
        <div id="content">Main content</div>
        <div id="right">Right sidebar</div>
    </div>
    <div id="footer">Footer Section</div>
</div>
```

这里唯一需要注意的就是，`footer`容器是被`container`容器包含在内的。

接着看css代码，

```css
html,body {
  margin: 0;
  padding:0;
  height: 100%;
}
#container {
  min-height:100%;
  height: auto !important;
  height: 100%; /*IE6不识别min-height*/
  position: relative;
}
#header {
    background: #ff0;
    padding: 10px;
}
#page {
    width: 960px;
    margin: 0 auto;
    padding-bottom: 60px;/*等于footer的高度*/
}
#footer {
    position: absolute;
    bottom: 0;
    width: 100%;
    height: 60px;/*脚部的高度*/
    background: #6cf;
    clear:both;
}
/*=======主体内容部分省略=======*/
```

从css代码中，我们看到，页面主体`page`设置了一个`padding-bottom`，并且与`footer`的高度是一致的。这里不能使用`margin-bottom`来代替`padding-bottom`。

完整的jsfiddle实例在[这里](http://jsfiddle.net/gejiawen/19bjc7yz/)。

这个方案有一个缺点：`footer`必须要固定高度，`page`必须要设置一个大于等于这个高度的`padding-bottom`。如果`footer`不是固定高度的，或者需要对footer做自适应，那么这种方案就不太适合了。

# 第二种方法

第二种方法的原理是：

> `footer`容器与`container`容易不再有包含关系，两者是同级的。给`html`，`body`，`container`容器的高度都设为100%，即`container`已经占据满了整个页面了，此时再添加`footer`容器，则`footer`必定会超出页面底部，而且会让页面出现滚动条。所以，我们给`footer`添加一个负值的`margin-top`，将`footer`容器从屏幕外拉上来。这个负值的`margin-top`与`footer`的高度相同。

我们先来看下html结构，

```html
<div id="container">
    <div id="header">Header Section</div>
    <div id="page" class="clearfix">
        <div id="left">Left sidebar</div>
        <div id="content">Main content</div>
        <div id="right">Right sidebar</div>
    </div>
</div>
<div id="footer">Footer section</div>
```

这里可以看出，`footer`容器和`container`容器是同级关系。

再看css代码，

```css
html,
body {
    height: 100%;
    margin: 0;
    padding: 0;
}
#container {
    min-height: 100%;
    height: auto !important;
    height: 100%;
}
#page {
    padding-bottom: 60px; /*高度等于footer的高度*/
}
#footer {
    position: relative;
    margin-top: -60px; /*等于footer的高度*/
    height: 60px;
    clear:both;
    background: #c6f;
}
/*==========其他div省略==========*/
```

我们给`footer`容器设置了一个**负值**的`margin-top`，目的就为了将`footer`容器从屏幕外拉上来。给`page`容器添加`padding-bottom`的目的是为了将`footer`容器的高度计算在`page`的大小内，这样当页面出现滚动条的行为才会正确。

完整的jsfiddle实例在[这里](http://jsfiddle.net/gejiawen/4npL8uLr/)。

这个方案的缺点跟第一种方法是一样的。

# 第三种方法

第三种方法的原理是，

> 使用一个无内容的`push`容器把`footer`容器推在页面最底部，同时还要给`container`设置一个负值的`margin-bottom`，这个`margin-bottom`与`footer`容器和`push`容器的高度是一致的。其实这种方法跟第二种方法是比较接近的。不过它多了一个无意义的`push`容器。

我们来看下相关的html结构，

```html
<div id="container">
    <div id="header">Header Section</div>
    <div id="page" class="clearfix">
        <div id="left">Left sidebar</div>
        <div id="content">Main Content</div>
        <div id="right">Right Content</div>
    </div>
    <div class="push"><!-- not put anything here --></div>
</div>
<div id="footer">Footer Section</div>
```

css代码，

```css
html,
body{
    height: 100%;
    margin:0;
    padding:0;
}
#container {
    min-height: 100%;
    height: auto !important;
    height: 100%;
    margin: 0 auto -60px;/*margin-bottom的负值等于footer高度*/
}
.push,
#footer {
    height: 60px;
    clear:both;
}
/*==========其他div效果省略==========*/
```

完整的jsfiddle实例在[这里](http://jsfiddle.net/gejiawen/om5uf2hu/)。

缺点：较之前面的两种方法，多使用了一个`div.push`容器，同样此方法限制了`footer`部分高度，无法达到自适应高度效果。

# 总结

前面介绍的三种方法都是采用css以及配合特定的html结构来达到的。这种方式比较轻量，在新版本的现代浏览器上都能够表现良好。不过使用css这种方式都必须要求`footer`的高度是固定的，因为页面主体容器主要就是对这个footer高度作手脚来达到页脚始终固定在底部的目的。

除了使用css的方式之外，还有一种**快糙猛**的方式，那就直接使用js代码来操作dom元素。这种方式对html不作限制，而且理论上能够兼容所有浏览器。不过这种方法我个人不是很推荐，因为直接使用js来操作dom是一个很**重**的行为，不利于html、css的表现，而且当页面发生resize时，页面由于重绘往往会导致一些闪烁或者卡顿。

# 参考列表

- [怎么使用Sticky Footer代码(让页脚紧贴页面底部的方法)](http://www.cnblogs.com/rippleyong/archive/2009/11/05/1596526.html)
- [Exploring Footers](http://alistapart.com/article/footers)
- [Make the Footer Stick to the Bottom of a Page](http://ryanfait.com/resources/footer-stick-to-bottom-of-page/)
- [如何将页脚固定在页面底部](http://www.w3cplus.com/css/css-sticky-foot-at-bottom-of-the-page)

End. All rights reserved `@gejiawen`.
