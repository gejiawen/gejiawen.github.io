title: 浅谈CSS中的伪元素和伪类
date: 2014-11-20 15:32:33
categories: [CSS]
tags: [CSS拾遗系列, css科普]
---

在CSS中，模式匹配（pattern match）规则决定文档树上的元素使用究竟哪个样式规则。这个模式就叫做选择器（selector）。

# CSS选择器

CSS中的元素选择器除了可以id（`#`）、class（`.`）、属性（`[]`）选取元素以外，还有很重要的一类，就是根据元素的状态来选取元素，包括伪类（pseudo-class）和伪元素(pseudo-element)。

这些传统的选择器，包括**id选择器**、**class选择器**、**属性选择器**、**派生选择器**等等，他们是直接从HTML文档的DOM树中获取元素的。而**伪类**和**伪元素**选择器是预定义的，且是独立文档元素的。它们获取元素的途径也不是基于id、class、属性这些基础的元素特征，而是基于**处于特殊状态的元素（伪类）**，或者是**元素中特别的内容（伪元素）**。当然，伪类和伪元素的表示形式也使用`:`（或者`::`）与其它选择器相区分。

# CSS伪类

什么叫伪类呢？

**伪类**是基于元素的*特征*而不是他们的id、class、属性或者内容。一般来说，元素的特征是不可以从DOM树上推断得到的，而且其是动态的，当用户和DOM进行交互的时候，元素可以获得或者失去一个伪类。（这里有一个例外，就是`:first-child`和`:lang`是可以从DOM树中推断出来的。）

CSS的现有标准中，伪类包括：

- `:first-child`  [点我看手册](http://www.w3school.com.cn/cssref/selector_first-child.asp)，应用第一个子元素
- `:link`  [点我看手册](http://www.w3school.com.cn/cssref/selector_link.asp)，应用未访问过的链接
- `:visited`  [点我看手册](http://www.w3school.com.cn/cssref/selector_visited.asp)，应用已访问过的链接
- `:hover`  [点我看手册](http://www.w3school.com.cn/cssref/selector_hover.asp)，应用鼠标指针悬浮的元素
- `:active`  [点我看手册](http://www.w3school.com.cn/cssref/selector_active.asp)，应用活动的链接
- `:focus`  [点我看手册](http://www.w3school.com.cn/cssref/selector_focus.asp)，应用聚焦的输入元素
- `:lang`  [点我看手册](http://baike.baidu.com/view/6763494.htm?fr=aladdin)，伪类将应用于元素带有指定lang的情况，不常用

# CSS伪元素

什么是伪元素呢？

**伪元素**是创造文档树之外的对象。例如文档不能提供访问元素内容第一字或者第一行的机制。伪元素还提供一些在源文档中不存在的内容分配样式，例如`:before`和`:after`能够访问产生的内容。伪元素的内容实际上和普通DOM元素是相同的，但是它本身只是基于元素的抽象，并不存在于文档中，所以叫伪元素。

CSS的现有标准中，伪元素包括：

- `:first-letter`  [点我看手册](http://www.w3school.com.cn/cssref/selector_first-letter.asp)，伪元素的样式将应用于元素文本的第一个字（母）
- `:first-line`  [点我看手册](http://www.w3school.com.cn/cssref/selector_first-line.asp)，伪元素的样式将应用于元素文本的第一行
- `:before`  [点我看手册](http://www.w3school.com.cn/cssref/selector_before.asp)，在元素内容的最前面添加新内容
- `:after`  [点我看手册](http://www.w3school.com.cn/cssref/selector_after.asp)，在元素内容的最后面添加新内容

# 两者之间区别

首先说一下**伪类**和**伪元素**的相同之处，

> 伪类和伪元素都不出现在源文件和文档树中。也就是说在html源文件中是看不到伪类和伪元素的。

他们的不同之处，

> 伪类其实就是基于普通DOM元素而产生的不同状态，他是DOM元素的某一特征。伪元素能够创建在DOM树中不存在的抽象对象，而且这些抽象对象是能够访问到的。

一句话总结不同之处就是，**伪元素产生新对象，在DOM树中看不到，但是可以操作；伪类不产生新的对象，仅是DOM中一个元素的不同状态；**

# `:before`和`:after`使用场景

现在在一些主流的css框架中，比如[Bootstrap](http://www.bootcss.com/)，[Foundation](http://foundation.zurb.com/)等中，对`:before`及`:after`的使用较多，而且这两个伪元素在一些特定场景下的确有许多妙用。

[demo1](http://www.jiawin.com/css-before-after/)是一个专门介绍使用`:before`及`:after`的博文，可以学习下。

下面，说一下我之前使用`:before`及`:after`的一个场景。

现在我们需要使用纯CSS做一个下图中的镂空箭头符号，

![](http://7xkwt1.com1.z0.glb.clouddn.com/CSS中的伪元素和伪类-001.jpg)

对应的html代码如下，

```html
<div class="arrow arrow-top"></div>
<div class="arrow arrow-right"></div>
<div class="arrow arrow-bottom"></div>
<div class="arrow arrow-left"></div>
```

样式如下，

```css
div.arrow:before, div.arrow:after {
    content: ' ';
    height: 0;
    width: 0;
    top: 100px;
    left: 255px;
    position: absolute;
    border: 10px solid transparent;
}

div.arrow-top:before {
    border-bottom-color: #fff;
    z-index: 2;
    top: 102px;
}

div.arrow-top:after {
    border-bottom-color: #333;
    z-index: 1;
}

div.arrow-right:before {
    border-left-color: #fff;
    z-index: 2;
    left: 297px;
    top: 104px;
}

div.arrow-right:after {
    border-left-color: #333;
    z-index: 1;
    left: 300px;
    top: 104px;
}

div.arrow-bottom:before {
    border-top-color: #fff;
    top: 107px;
    left: 330px;
    z-index: 2;
}

div.arrow-bottom:after {
    border-top-color: #333;
    top: 110px;
    left: 330px;
    z-index: 1;
}

div.arrow-left:before {
    border-right-color: #fff;
    top: 103px;
    left: 355px;
    z-index: 2;
}

div.arrow-left:after {
    border-right-color: #333;
    top: 103px;
    left: 352px;
    z-index: 1;
}
```

其实原理很简单，

> 设置`.arrow`属性的`:before`和`:after`的`border`属性为`10px`，颜色为透明的。然后将`:before`和`:after`中的任意一层的`border-color`设置为可辨识的，然后使用`z-index`值较高的层遮盖`z-index`值较低的层，通过微调`top`和`left`的值达到目的。

这里我们当然可以通过一些美化的手段，使得我们的箭头看起来更加好看一点，比如像下面这样，

![](http://7xkwt1.com1.z0.glb.clouddn.com/CSS中的伪元素和伪类-002.jpg)


# 参考列表

- [W3School](http://www.w3school.com.cn/cssref/index.asp)
- [css伪类伪元素](http://www.php100.com/html/webkaifa/DIV_CSS/2008/0820/2231.html)
- [CSS 伪类与伪元素](http://blog.csdn.net/sadfishsc/article/details/7047595)


End. All rights reserved `@gejiawen`.


