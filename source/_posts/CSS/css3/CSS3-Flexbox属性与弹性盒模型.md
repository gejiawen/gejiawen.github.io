postid: "css3-flexbox-guide"
title: "CSS3 Flexbox属性与弹性盒模型"
date: 2015-05-20 18:12:00
categories: [CSS]
tags: [CSS拾遗系列, css3]

---

# 简介

CSS3新增的[Flexible Box Layout](http://www.w3.org/TR/css-flexbox/)（弹性盒模型布局方式）是CSS3规范中提出的一种新的布局方案。该布局方案提供了一种更加高效简单的方式来处理容器中的元素布局、对齐、空间分配等操作，即使容器中的元素尺寸未知（或者尺寸大小是动态的）也能工作得很好。目前CSS3提出的此种布局方式也被各大主流浏览器所支持，可以预见Flexbox Layout在未来将会被广泛使用。

本文将详细的介绍该布局模型，同时会辅以一定的示例来具体说明如何在具体的web开发中应用该布局模型，达到传统布局方案（块级布局、内联布局、绝对定位局部等）的效果。

# 初识Flexbox

先让我们来说一说Flexbox Layout（弹性盒模型布局）的基本原理。

在弹性盒模型布局中，需要事先指定一个容器，后续的所有布局操作都是基于此容器来定义的。其核心是**容器会根据布局的需要（动态的）调整其中所包含的子元素（即布局条目）的尺寸、顺序来填充容器的所有可用空间。**

当容器的尺寸由于屏幕大小（或者浏览器窗口尺寸）发生变化时，其中包含的布局条目也会自动地进行调整。举个例子，当容器尺寸增大时，包含的条目将会自动拉伸以沾满多余的空白空间；当容器尺寸变小时，条目会自动收缩以适应容器的尺寸防止移除容器的范围。

额外提一点，Flexbox布局是**不存在内置的布局方向**的。这是什么意思呢？比如传统的布局方案中，块级布局（block）默认是将各个块级元素按照垂直方向自上向下依次堆放；内联布局（inline）默认是将各个内联元素按照水平方向按照从左到右的顺序依次堆放。而弹性盒布局不存在这种默认的布局方向限制，它提供了独立的布局方向属性，开发人员可以根据布局需要自行设置。

## 相关概念

弹性盒布局是一种新的布局方式。它涉及到一些新的概念，这里我们对其加以解释。如下图所示，

![](http://7xkwt1.com1.z0.glb.clouddn.com/CSS3-Flexbox属性与弹性盒模型-001.png)

它包含如下几个概念，

1. `flex container`，即所谓的**容器**（或者称之为**弹性盒**、**flex容器**）。注意这里说的容器并不是单纯的dom元素所指代的容器，而是特指设置了弹性盒布局的dom元素。
2. `flex item`，即所谓的**条目**（或者称之为**flex条目**）。这里的条目其实就是容器的子元素。比如`ul`元素为一个flex容器，那么其内部包含的`li`元素就是flex条目。大部分时候，不同的弹性盒布局真正产生变化的其实就是条目的布局行为发生了变化。
3. `main axis`和`cross axis`，即所谓的**主轴**和**交叉轴**（有的翻译文章也称之为**侧轴**）。这两个属性定义flex布局方向。需要注意的是，虽然图中主轴是水平方向，交叉轴是垂直方向，但是这并不是固定的，开发人员完全自定义主轴和交叉轴的方向。
    1. 在使用flex布局时，一般我们需要首先明确主轴的方向，然后交叉轴的方向也会随之确定下来。因为**交叉轴的方向始终是与主轴的方向是垂直的**。
    2. flex容器中flex条目可以根据布局需要排列成**单行**或者**多行**。
    3. 主轴`main axis`的作用是确定每一行上flex条目的排列方向。
    4. （当flex条目成多行排列时）交叉轴`cross axis`的作用是确定行与行之间的排列方向。
4. `main start`和`main end`，即所谓的**主轴起点**和**主轴终点**。
    1. 明确主轴的方向后（如上所述，不管是主轴还是交叉轴其实只有两种方向，即水平或者垂直），还需要确定他们各自的排列方向。
    2. 假如已经明确主轴的方向是水平的，那么其排列方向仍然会有两种可选，一种是**从左到右**的排列方向（flex条目从左到右依次堆放），另一种是**从右到左**排列方向（flex条目从右到左依次堆放）。
    3. 主轴起点在左主轴终点在右即为**从左到右**的排列方向。（如上图所示）
    4. 主轴起点在右主轴终点在左即为**从右到左**的排列方向。
5. `cross start`和`cross end`，即所谓的**交叉轴起点**和**交叉轴终点**。
    1. 跟主轴起点和主轴终点的含义类似，交叉轴起点和交叉轴终点明确了行与行之间的排列顺序。
    2. 假如已经明确交叉轴的方向是垂直的，那么其排列方向仍然将会有两种可选，一种是**从上到下**的排列方向（第二行在第一行的下方），另一种是**从下到上**的排列方向（第二行在第一行的上方）。
6. 所以，在flex容器进行布局时，在每一行中会把其中的flex条目从主轴起始位置开始，依次排列到主轴结束位置。当flex容器中存在多行时，会把每一行从交叉轴起始位置开始，依次排列到交叉轴结束位置。
7. `main size`和`cross size`，即所谓的**主轴尺寸**和**交叉轴尺寸**。对应dom元素在主轴和交叉轴上的大小。
    1. 如果主轴方向是水平的（那么交叉轴方向肯定是垂直的），此时主轴尺寸即是dom元素（flex容器）的宽度属性，交叉轴尺寸即是dom元素（flex容器）的高度属性。
    2. 如果主轴方向是垂直的，那么主轴尺寸和交叉轴尺寸对应的dom元素宽高属性与之前相反。

继续学习flex布局之前，一定要弄明白这些概念，因为具体的属性设置会与这些概念息息相关。  

## 新旧语法

在进行flex布局相关属性的详细说明之前，有必要提一点。就是flex布局先后有两种语法，一种是W3C于2009年发布的旧语法，另一种是W3C最新发布的新语法。

这里我不对前后两种语法的差异作解析，仅从语法层面说明存在这个差异。

旧语法中，所有的flex属性都以`box`打头，flex设置为`display: box`。如下图，

![](http://7xkwt1.com1.z0.glb.clouddn.com/CSS3-Flexbox属性与弹性盒模型-002.png)

新语法中，所有的flex属性都以`flex`打头，flex设置为`display: flex`。如下图，

![](http://7xkwt1.com1.z0.glb.clouddn.com/CSS3-Flexbox属性与弹性盒模型-003.png)

关于这两种更多的差异请参阅官方文档或者[CSS参考手册](http://css.doyoe.com/)。

其中新旧语法中间还有存在一种非官方的过渡语法规范，flex设置为`display: flexbox`

顺便提一句，关于新旧语法在实际使用时，肯定是推荐使用新语法的，虽然新语法的浏览器支持性不如旧语法，但是毕竟是旧的语法，迟早是要淘汰的，而且新语法的表现也更加一致。

# 属性详解

**注意**，flex布局中的css样式声明与一般的css声明不同，它会有两个作用对象，分别是**flex容器**和**flex条目**。即，有的flex属性只能作用于flex容器，而有的flex属性只能作用于flex条目。

下面我们针对flex布局的各个属性分别作详细说明。

## 问题引导

在详细阐述各个属性之前，我们先来抛出一个比较常见的布局问题。问题是这样的，现在有一个无序列表（`ul`元素），其包含了6个列表项（`li`元素），如何做可以让列表项（`li`元素）自适应父容器的尺寸？换句话说，当父容器尺寸足够大时，所有的`li`元素平铺成一行；当父容器的尺寸减小时，`li`元素自动换行。

大家都能想到的一种常规做法应该就是给`li`元素设置浮动。不过这里我们将使用flex布局来实现这种布局需求。html和css代码如下，

```html
<ul class="flex-container">
    <li class="flex-item">1</li>
    <li class="flex-item">2</li>
    <li class="flex-item">3</li>
    <li class="flex-item">4</li>
    <li class="flex-item">5</li>
    <li class="flex-item">6</li>
</ul>
```

```css
.flex-container {
    list-style: none;
    display: flex;
    flex-direction: row;
    flex-wrap: wrap;
}
.flex-item {
    margin: 5px;
    width: 300px;
    height: 300px;
    text-align: center;
    line-height: 300px;
    background-color: gold;
}
```

具体的效果可以看这个[demo](http://runjs.cn/detail/yzwrgtml)。各位看官可以自行缩放浏览器的窗口，观察`.flex-item`块的排列行为。注意，这里我并没有使用浮动的相关属性。

## `display: flex`

在之前说过，要想使用弹性盒布局必须事先指定一个容器，这个容器就是所谓的flex容器。那么如何指定flex容器呢？很简单，将其`display`属性设置为`flex`即可。如下，

```css
.flex-container {
    display: flex;
}
```

**注意**，如果需要在内联的场景下使用flex布局，则需要设置`display: inline-flex`。而且以下几种属性设置在弹性盒布局中是不起作用的，

1. 浮动元素（`float`）
2. 清除浮动（`clear`）
3. css3多列布局（`columns-*`）
4. 垂直居中（`vertical-align`）
5. 首行/首字符选择伪类（`::first-line`和`::first-letter`）

## `flex-direction`

`flex-direction`属性的作用是设置**主轴**方向。我们之前有说过，一旦主轴方向确定，那么连带的交叉轴的方向也会确定下来。这样就确定了flex条目基本的排列方式。

下面我们来看一下`flex-direction`的具体用法，如下表，

| 可选值 | 含义 | 备注 |
| :---  | :--- | :--- |
| `row` | 横向从左到右排列 | 即**左对齐**。此为默认值 |
| `row-reverse` | 反转横向排列（从右到左拍立） | 即**右对齐** |
| `column` | 纵向排列（从上到下排列） | 即设置主轴方向为垂直方向 |
| `column` | 反转纵向排列（从下到上排列） | 主轴方向为垂直方向，但是越是后面的flex条目反而在上方 |

在这个[demo](http://runjs.cn/detail/yzwrgtml)中，可以肆意改变`flex-direction`的值，并改变浏览器窗口大小，观察页面布局的变化。

## `flex-wrap`

`flex-wrap`属性用于设置当所有的flex条目的尺寸之和超过flex容器的主轴尺寸时应该采取的行为。这是什么意思呢？

在默认情况下，flex容器中的flex条目会尽量的在一行上占满flex容器的空间。当主轴尺寸大于所有flex条目尺寸之和时，这时并没有什么问题。但是，当主轴尺寸小于所有flex条目尺寸之和时，此时就会出现flex条目之间相互重叠或者换行的情况。所以`flex-wrap`属性就是专门用来处理这种情况的。

下面让我们来看一下`flex-wrap`的具体用法，如下表，

| 可选值 | 含义 | 备注 |
| :--- | :--- | :--- |
| `nowrap` | 容器中的条目只占满容器在主轴方向上的一行，此时可能出现条目互相重叠或超出容器范围的现象 | 此为默认值 |
| `wrap` | 当容器中的条目超出主轴方向上的一行时，会把条目排列到下一行。而下一行的位置与交叉轴的方向一致 | 交叉轴的存在感就是在此 |
| `wrap-reverse` | 与`wrap`的含义类似，不同的是下一行的位置与交叉轴的方向相反。 | - |

与之前类似，我们可以在这个[demo](http://runjs.cn/detail/yzwrgtml)中实验不同`flex-wrap`取值的差异。

## `flex-flow`

之前说的两个属性`flex-direction`和`flex-wrap`，从某种意义上来说就是设置flex布局的主轴和交叉轴。一旦flex布局的主轴和交叉轴确定下来，基本上整个布局中flex条目的排列方式我们就可以自行脑补出来了。

而`flex-flow`属性其实就是`flex-direction`和`flex-wrap`的复合属性。所以下面两段代码的效果是一致的。

```css
.flex-container1 {
    flex-direction: row;
    flex-wrap: wrap;
}

.flex-container2 {
    flex-flow: row wrap;
}
```

## `order`

前面说的几个属性其实都是针对flex容器的设置。而`order`属性是针对flex条目的。它的作用是自定义flex容器中条目的顺序。其用法示例如下，

```css
.flex-item:last-child {
    order: -1;
}
```

这段css代码中使用了css3新出的一个**结构伪类选择器**`:last-child`，其作用是匹配`.flex-item`元素且该元素是其父元素的最后一个子元素。关于css3选择器的更多内容欢迎阅读博主的这篇文章[初识CSS3选择器](http://blog.gejiawen.com/2015/04/09/first-to-css3-selectors/)。

言归正传，上面的css代码，将导致最后一个`li`元素反而在第一个展示。效果如下，

![](http://7xkwt1.com1.z0.glb.clouddn.com/CSS3-Flexbox属性与弹性盒模型-004.png)

**注意**，此时虽然最后一个子元素被置顶展示，但是并没有改变其html文档结构。

`order`的用法如下，

```
order: <integer>
```

用整数值来定义排列顺序，数值小的排在前面。默认值为0，且可以设置为负值。

## flex条目的弹性设置

弹性盒模型的真正牛逼的地方在于flex容器中的**flex条目的尺寸是弹性的**。这是啥意思呢？个人觉得它包含下面的几层含义，

- 主轴尺寸不够大时，使用`flex-wrap`可以让flex条目自动换行。
- 主轴尺寸不够大时，还可以缩小条目的尺寸防止溢出容器范围。
- 主轴尺寸足够大时，可以适当扩展条目的尺寸来占用容器的额外空白空间。

这里我们把重心放在后两点上，即**flex容器可以根据本身尺寸的大小来动态地调整flex条目的尺寸**。

flex条目尺寸的弹性由3个css属性来确定，分别是`flex-basis`、`flex-grow`和`flex-shrink`。他们的相关说明如下，

| 属性 | 可选值 | 默认值 | 作用对象 | 含义 | 备注 |
| :--- | :--- : | :--- | :---   | :--- | :--- |
| `flex-basis` | `<length>`, `<percentage>`, `auto` | `auto` | flex条目 | 设置弹性条目的初始主轴尺寸 | 这里的初始值表示flex容器调整之前flex条目尺寸的初始值。此属性跟`width`等属性的设置模式是一致的 |
| `flex-grow` | `<integer>` | `1` | flex条目 | 当容器有多余的空间时，这些空间在不同条目之间的分配比例 | 用于设置flex条目的扩展因子，不能取负值 |
| `flex-shrink` | `<integer>` | `1` | flex条目 | 当容器的空间不足时，各个条目尺寸缩小的比例 | 此属性的作用与`flex-grow`相反 |

关于上述三个属性的一些额外说明，

### `flex-basis`

`flex-basis`用于设置flex条目的初始尺寸（未进行任何调整之前）。当设置为`auto`时，则实际使用的值是主轴尺寸属性的值。若主轴尺寸也是`auto`，那么使用的值由条目内容的尺寸来确定。

### `flex-grow`

当主轴尺寸足够大时，flex条目在容器内一行就全部排列完了，此时容器的空间还有剩余，那么可用`flex-grow`扩展flex条目。举个例子，一个容器中有3个flex条目，其`flex-grow`属性分别如下，

```css
.flex-item:nth-child(1) {
    flex-grow: 1;
}
.flex-item:nth-child(2) {
    flex-grow: 2;
}
.flex-item:nth-child(3) {
    flex-grow: 3;
}
```

同时，flex容器的宽度是800px，每个条目的宽度为200px，所以容器还将剩余200px。由于条目都设置了`flex-grow`属性，那么此三个条目将按比例分配容器的剩余空间。第一个条目将得到200 * 1/6 = 33px左右，第二个条目将得到200 * 2/6 = 66px，第三个条目将得到200 * 3/6 = 99px左右。效果如下图，

![](http://7xkwt1.com1.z0.glb.clouddn.com/CSS3-Flexbox属性与弹性盒模型-005.png)

详情可参考这个[demo](http://runjs.cn/detail/ysrnndee)。

### `flex-shrink`

`flex-shrink`在使用上类似`flex-grow`，不过它使用于设置当主轴尺寸不够大时缩小flex条目。同样的，我们来举个例子，一个容器中有3个flex条目，每个felx条目的尺寸为200px，容器的尺寸为500px。很明显此时容器尺寸已经放不下其中三个flex条目了。我们对每个flex条目设置`flex-shrink`属性，

```css
.flex-item:nth-child(1) {
    flex-shrink: 1;
}
.flex-item:nth-child(2) {
    flex-shrink: 2;
}
.flex-item:nth-child(3) {
    flex-shrink: 3;
}
```

如果我们希望flex容器能够刚刚好将三个flex条目放下，我们需要缩小的尺寸为600-500 = 100px。接下来三个条目将根据`flex-shrink`属性按比例的分摊需要缩小的尺寸，即分别为100 * 1/6 = 16.6px，100 * 2/6 = 33.3px，100 * 3/6 = 50px。所以三个flex条目最终的尺寸为200 - 16.6 = 183.4px，200 - 33.3 = 166.7px，200 - 50 = 150px。效果如下，

![](http://7xkwt1.com1.z0.glb.clouddn.com/CSS3-Flexbox属性与弹性盒模型-006.png)

详情可参考这个[demo](http://runjs.cn/detail/orbwqkkp)。

### 多行弹性布局的注意事项

**⚠️注意**，这里额外提一个非常重要的点，就是在我们使用`flex-grow`和`flex-shrink`属性对flex条目进行扩展或者缩小时，这些操作其实是以**行**为单位来操作的，容器先根据`flex-wrap`的属性值来确定是单行布局或多行布局，然后把条目分配到对应的行中，最后在每一行内进行空白空间的分配。

这什么意思呢？我们来举个简单例子说明一下。假如我有一份代码如下，

```html
<style>
.flex-container {
    width: 999px;
    flex-direction: row;
    flex-wrap: wrap;
    display: flex;
}
.flex-item {
    width: 300px;
    flex-grow: 1;
    flex-shrink: 1;
    flex-basis: auto;
}
</style>

<ul class="flex-container">
    <li class="flex-item">1</li>
    <li class="flex-item">2</li>
    <li class="flex-item">3</li>
    <li class="flex-item">4</li>
</ul>
```

这里容器中有4个flex条目，每个条目的宽度为300px，但是容器的尺寸只有999px，明显是不足以放下所有的flex条目的。此时前三个条目将堆放在第一行，第4个条目将单独的被堆放在第二行。此时第一行将会多出99px的空余空间，这99px的空余空间将会平均应用到第一行的三个条目上；第二行因为只有一个条目，此时这个条目将会直接占用剩余的699px。整个效果如下图，

![](http://7xkwt1.com1.z0.glb.clouddn.com/CSS3-Flexbox属性与弹性盒模型-007.png)

详情可参考这个[demo](http://runjs.cn/detail/zabhpmlq)。

## `flex`复合属性

如你所见，`flex`是一个复合属性，它的语法如下，

```
flex: none | auto | [flex-grow] | [flex-shrink] | [flex-basis]
```

其中`none`的含义为：`0 0 auto`, 即`flex-grow: 0`，`flex-shrink: 0`， `flex-basis: auto`。
其中`auto`的含义为：`1 1 auto`, 即`flex-grow: 1`，`flex-shrink: 1`， `flex-basis: auto`。

当`flex-basis`被省略时，其值为`0%`。

## flex条目的对齐

当flex容器中flex条目的尺寸和排列都确定下来之后，我们还可以设置这些flex条目在容器中的对齐方式。

目前有三种对齐方式。

### 基于`margin: auto`对齐

第一种对齐方式是**使用空白边**，即`margin: auto`。在使用自动空白边时，flex容器中额外的空白空间将会由被声明为`auto`的空白边占据。

这种方式在W3C的官方文档被称之为*non-normative*，更加像是一种奇淫巧计。

我们来看个具体的例子，

```html
<nav>
    <ul>
        <li><a href=/about>About</a>
        <li><a href=/projects>Projects</a>
        <li><a href=/interact>Interact</a>
        <li id="login"><a href=/login>Login</a>
    </ul>
</nav>
```

```css
ul {
    list-style: none;
    display: flex;
}
li {
    min-width: min-content;
}
#login {
   margin-left: auto;
}
```

其效果如下，

![](http://7xkwt1.com1.z0.glb.clouddn.com/CSS3-Flexbox属性与弹性盒模型-008.png)

示例中的`#login`元素通过简单的`margin-left`设置，将其置于最右侧展示。详情可参考这个[demo](http://runjs.cn/detail/i5k0lsn0)。

### 基于主轴对齐

第二种对齐方式是基于主轴方向上的对齐策略。可以通过`justify-content`属性来进行设置。借助此属性，我们可以调整flex条目在主轴方向上的对齐策略。

这种对齐方式的调整**发生在修改条目的弹性尺寸和处理自动空白边之后**。当容器中的一行中的条目没有弹性尺寸或是已经达到了它们的最大尺寸时，如果此时在这一行上还有额外的空白空间，我们可以使用`justify-content`来分配这些空白空间。该属性还可以控制当条目超出行的范围时的对齐方式。

我们来看下`justify-content`属性的基本用法，（**注意**，`justify-content`用于设置flex容器）

| 可选值 | 含义 |
| :---  | :--- |
| `flex-start` | 条目集中于该行的起始位置。第一个条目与其所在行在主轴起始方向上的边界保持对齐，其余的条目按照顺序依次排列。|
| `flex-end` | 条目集中于该行的结束方向。最后一个条目与其所在行在主轴结束方向上的边界保持对齐，其余的条目按照顺序依次排列。 |
| `center` | 条目集中于该行的中央。条目都往该行的中央排列，在主轴起始方向和结束方向上留有同样大小的空白空间。如果空白空间不足，则条目会在两个方向上超出同样的空间。 |
| `space-between` | 第一个条目与其所在行在主轴起始方向上的边界保持对齐，最后一个条目与其所在行在主轴结束方向上的边界保持对齐。空白空间在条目之间平均分配，使得相邻条目之间的空白尺寸相同。 |
| `space-around` | 类似于`space-between`，不同的是第一个条目和最后一个条目与该行的边界之间同样存在空白空间，该空白空间的尺寸是条目之间的空白空间的尺寸的一半。 |

各属性值的效果如下图，

![](http://7xkwt1.com1.z0.glb.clouddn.com/CSS3-Flexbox属性与弹性盒模型-009.png)

### 基于交叉轴对齐
 
 第三种对齐方式是基于交叉轴方向上的对齐策略。此种方式中，我们有两个属性可以做相关设置，它们分别是`align-items`和`align-self`。其中前者是用来设置flex容器的，后者是用来设置flex条目的，这两个属性的作用对象是不同的。在某些场景下，我们可以对flex条目设置`align-self`来复写flex容器指定的对齐方式。
 
 我们来看看`align-items`的基本用法，
 
 | 可选值 | 含义 |
 | :--- | :--- |
 | `flex-start` | 条目与其所在行在交叉轴起始方向上的边界保持对齐。 |
 | `flex-end` | 条目与其所在行在交叉轴结束方向上的边界保持对齐。 |
 | `center` | 条目的空白边盒子（margin box）在交叉轴上居中。如果交叉轴尺寸小于条目的尺寸，则条目会在两个方向上超出相同大小的空间。 |
 | `baseline` | 条目在基准线上保持对齐。在所有条目中，基准线与交叉轴起始方向上的边界距离最大的条目，它与所在行在交叉轴方向上的边界保持对齐。 |
 | `stretch` | 如果条目的交叉轴尺寸的计算值是`auto`，则其实际使用的值会使得条目在交叉轴方向上尽可能地占满。 |
 
 各属性值的效果如下图，
 
 ![](http://7xkwt1.com1.z0.glb.clouddn.com/CSS3-Flexbox属性与弹性盒模型-010.png)
 
 `align-self`的属性基本与`align-items`一致（用法和含义基本都一样）。不过`align-self`除了`align-items`属性可选值之外，还可以设置为`auto`。当设置为`auto`时，其值是父节点的属性`align-items`的值,如果该节点没有父节点，则为`stretch`。
 
## flex行的对齐
 
前面说完了flex容器中flex条目的对齐方式。这里我们再来说一下**flex行**的对齐。什么叫flex行？W3C官方文档中提到了一个名词，叫做[**Flex Lines**](http://www.w3.org/TR/css-flexbox/#flex-lines)。

那么什么叫Flex Lines呢？说的直白点就是多行flex布局时，主轴方向上的每一行就是Flex Lines。

举个例子，

```html
<div id="flex">
    <div class="item">1</div>
    <div class="item">2</div>
    <div class="item">3</div>
    <div class="item">4</div>
</div>
```

```css
#flex {
    display: flex;
    flex-flow: row wrap;
    width: 300px;
}
.item {
    width: 80px;
}
```

很明显，上述代码生成的flex容器一行肯定堆放不下所有flex条目的。又因为设置了`flex-wrap: wrap`，所以其在主轴方向上会发生换行。如下图，

![](http://7xkwt1.com1.z0.glb.clouddn.com/CSS3-Flexbox属性与弹性盒模型-011.png)

如图所示，此flex布局将分为两行，每一行就是所谓的**Flex Lines**。

如果一个flex布局容器中不止一行，我们同样可以对行与行之间的对齐方式进行设置。为这一设置提供支持的属性就是`align-content`。其实此属性的作用和`justify-content`属性很相似，只不过`justify-content`是用于在主轴方向上对齐行内的flex条目，而`align-content`是用于设置行与行之间的对齐策略。

`align-content`详细的用法如下表，

| 可选值 | 含义 |
| :--- | :--- |
| `flex-start` | 行集中于容器的交叉轴起始位置。第一行与容器在交叉轴起始方向上的边界保持对齐，其余行按照顺序依次排列。 |
| `flex-end` | 行集中于容器的交叉轴结束位置。第一行与容器在交叉轴结束方向上的边界保持对齐，其余行按照顺序依次排列。 |
| `center` | 行集中于容器的中央。行都往容器的中央排列，在交叉轴起始方向和结束方向上留有同样大小的空白空间。如果空白空间不足，则行会在两个方向上超出同样的空间。 |
| `space-between` | 行在容器中均匀分布。第一行与容器在交叉轴起始方向上的边界保持对齐，最后一行与容器在交叉轴结束方向上的边界保持对齐。空白空间在行之间平均分配，使得相邻行之间的空白尺寸相同。 |
| `space-around` | 类似于`space-between`，不同的是第一行条目和最后一个行目与容器行的边界之间同样存在空白空间，而该空白空间的尺寸是行目之间的空白空间的尺寸的一半。 |
| `stretch` | 伸展行来占满剩余的空间。多余的空间在行之间平均分配，使得每一行的交叉轴尺寸变大。 |

各属性值的效果如下，

![](http://7xkwt1.com1.z0.glb.clouddn.com/CSS3-Flexbox属性与弹性盒模型-012.png)

# 浏览器兼容性

本文的**新旧语法**中曾经提到，弹性盒模型官方有好几个版本的语法，这是因为其规范本身有过多个不同版本，因为浏览器对于该规范的支持也存在一些不一致。大致来说总共有三个不同版本的语法，

- 最新规范：最新版本规范的语法，即`display: flex`。
- 中间版本：2011年的非官方规范语法，即`display: flexbox`。
- 老规范：2009年的规范的语法，即`display: box`。

浏览器的支持情况如下，

| Chrome | Safari | Firefox | Opera | IE | Android | iOS |
| :---   | :---   | :---    | :---  | :--- | :---  | :--- |
| 21+（新规范） | 6.1+（新规范） | 22+（新规范） | 12.1+（新规范） | 11+（新规范）| 4.4+（新规范） | 7.1+（新规范）|
| 20-（老规范）| 3.1+（老规范）| 2-21（老规范）| - | 10（中间版本）| 2.1+（老规范）| 3.2+（老规范）|


从上面看来，弹性盒布局模型基本已经被主流的现代浏览器所支持。除了IE系的浏览器拖了后腿之外，基本可以无障碍的使用。不过虽然如此，我们为了兼容性，在实际使用的时候除了规范中定义的属性之外，最好添加不同浏览器内核的私有前缀。

# 参考列表

- [w3c](http://www.w3.org/TR/css-flexbox/)
- [css3-flexbox](http://www.w3.org/html/ig/zh/wiki/Css3-flexbox)
- [深入理解 CSS3 弹性盒布局模型](http://www.ibm.com/developerworks/cn/web/1409_chengfu_css3flexbox)
- [CSS3 Flexbox](http://www.html-js.com/article/CSS-learning-CSS3-Flexbox)
- [详解css3弹性盒模型（Flexbox）](http://segmentfault.com/a/1190000000707526)
- [A Complete Guide to Flexbox](https://css-tricks.com/snippets/css/a-guide-to-flexbox/)




