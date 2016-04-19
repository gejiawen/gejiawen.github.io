postid: "css3-gradient-guide"
title: "CSS3渐变"
date: 2015-04-23 20:22:10
categories: [CSS]
tags: [CSS拾遗系列, css3]

---

**⚠️注意**：由于CSS3中的部分内容尚未完全定稿，所以本文的部分可能会随时更新。

# 缘起

在之前的一篇文章[CSS3背景](http://blog.gejiawen.com/2015/04/21/css3-background-guide/)中，我们有谈到`background-image`属性，此属性用于设置元素的背景图片，我们可以为之赋值一个图片url，还可以使用CSS3提供的渐变（gradient）技术来设置一张渐变的纯CSS背景。

CSS3支持的渐变可分为线性渐变（linear-gradient）、径向渐变（radial-gradient）、重复线性渐变（repeating-linear-gradient）和重复径向渐变（repeating-radial-gradient）。

# 兼容性

由于W3C官方组织尚未对渐变这一块完全定稿，虽然主流的现代浏览器基本都已经对渐变提供了支持，但是具体的实现细节可能不太一致，需要针对不同的浏览器内核添加私有前缀。

下图是摘自[www.w3cschool.cc](http://www.w3cschool.cc/)的浏览器兼容性图表，可供参考。

![](http://7xkwt1.com1.z0.glb.clouddn.com/CSS3渐变-001.png)

# 线性渐变

## 定义与语法

什么叫线性渐变？

颜色沿着一条直线过渡，比如从左侧到右侧、从右侧到左右、从顶部到底部、从底部到顶部、从左上角到右下角等等，简单来说渐变方向是一条直线，此直线可以沿着任意轴。如果有过PS、flash等软件或者设计经验的话，应该不难理解。

W3C规定的线性渐变语法如下，

```
background-image: linear-gradient(direction, color-start, ..., color-end);
```

即，`linear-gradient`包含如下几个基本参数，

- 第一个参数为**渐变方向**
- 第二个参数为**渐变起点**
- 第三个参数为**渐变终点**

同时，你还可以在渐变起点和渐变终点中间插入任意多个色标来达到多重颜色渐变的目的。

### 渐变方向

关于**渐变方向**，我们可以设置为一些常用关键词，比如top、right、bottom、left，以及这四个基本关键词的任意相邻方向的**两两组合**，比如top left、top right等等。

渐变方向除了可以设置为常用关键词之外，我们还可以设置一个角度值，即`0deg`～`360deg`。这个角度值也可以为负值。正负值的区别在于渐变方向相对于默认值的方向是不同的。

**⚠️注意**，此参数可以省略。当渐变方向省略时，W3C规定的渐变方向默认值为**top**，即渐变方向从上到下。

### 渐变起点

线性渐变的**渐变起点**其实是一个颜色值（后面说到的径向渐变也有渐变起点的概念，但是此起点非彼起点），而且还可以包含一个长度值，此长度值可省略。长度值存在的含义是，可以对渐变起点进行偏移。

### 渐变终点

**渐变终点**为一个颜色值，也可以包含一个长度值，此长度值可省略。长度值存在的含义是，可以对渐变终点进行偏移。

值得一提的是，**渐变终点**的偏移效果与**渐变起点**的偏移效果是不一样。欲知详情，请继续往下看😄。

## 用法

### 基本用法

下面有一段示例，让我们一起来体验一下（为了方便，css部分的代码仅给出了webkit内核的声明）

```html
<style>
div {
    width: 400px;
    height: 300px;
    border: 1px solid;
    margin-right: 15px;
    float: left;
}
.div1 {
    background-image: -webkit-linear-gradient(top, pink, green);
}
.div2 {
    background-image: -webkit-linear-gradient(top left, pink, green);
}
.div3 {
    background-image: -webkit-linear-gradient(45deg, pink, green);
}
</style>

<div class=“gradient div1”>
    -webkit-linear-gradient(top, pink, green);
</div>
<div class=“gradient div2”>
    -webkit-linear-gradient(bottom right, pink, green);
</div>
<div class=“gradient div3”>
    -webkit-linear-gradient(45deg, pink, green);
</div>
```

其效果如下图所示，

![](http://7xkwt1.com1.z0.glb.clouddn.com/CSS3渐变-002.png)

### 更多用法

上面我们已经看到了线性渐变的基本用法及其效果，下面我们来看一下针对不同参数进行不同赋值时的效果。

#### 示例1

首先，我们来尝试一下给**渐变起点**或者**渐变终点**设置一个长度值看看效果。代码如下，

```html
<style>
.gradient {
    width: 400px;
    height: 300px;
    font-size: 10px;
    margin-right: 15px;
    float: left;
    border: 1px solid;
    text-align: center;
    line-height: 80px;
}
.div1 {
    background-image: -webkit-linear-gradient(top, pink 100px, green 200px);
}
.div2 {
    background-image: -webkit-linear-gradient(bottom right, pink 100px, green 200px);
}
.div3 {
    background-image: -webkit-linear-gradient(45deg, pink, green 50%);
}
</style>
<div class=“gradient div1”>
    -webkit-linear-gradient(top, pink 100px, green 200px);
</div>
<div class=“gradient div2”>
    -webkit-linear-gradient(bottom right, pink 100px, green 200px);
</div>
<div class=“gradient div3”>
    -webkit-linear-gradient(45deg, pink, green 50%);
</div>
```

其效果如下，

![](http://7xkwt1.com1.z0.glb.clouddn.com/CSS3渐变-003.png)

从效果图中可以看出，

- **渐变起点**和**渐变终点**的长度值可以设置为CSS允许的长度单位（比如px、em等），还可以设置为百分比（％）
- 当给**渐变起点**设置长度时，渐变起点将会相对默认位置发生偏移。所设置的长度即为偏移值，而**偏移点为真正开始渐变的起点**
- 当给**渐变终点**设置长度时，渐变终点也将会发生偏移，**偏移点为真正的渐变终点**
- 当**渐变方向**即不是水平也不是垂直时（比如设置渐变方向为一角度值或者类似top left类似的值），偏移长度值与水平和垂直方向将组成一个三角形

#### 示例2

其次，将**渐变方向**设置为一个角度值时，我们可以给出一个负值，比如

```html
<style>
.gradient {
    width: 400px;
    height: 300px;
    font-size: 10px;
    margin-right: 15px;
    float: left;
    border: 1px solid;
    text-align: center;
    line-height: 80px;
}
.div1 {
    background-image: -webkit-linear-gradient(45deg, pink, green);
}
.div2 {
    background-image: -webkit-linear-gradient(-45deg, pink, green);
}
</style>
<div class=“gradient div1”>
    -webkit-linear-gradient(45deg, pink, green);
</div>
<div class=“gradient div2”>
    -webkit-linear-gradient(-45deg, pink, green);
</div>
```

其效果如下，

![](http://7xkwt1.com1.z0.glb.clouddn.com/CSS3渐变-004.png)

所以，当我们给**渐变方向**设置为一角度值时，其实是改变了渐变方向。

**⚠️注意**：此处在chrome（webkit内核）的浏览器上有一个坑。当我们设置渐变方向为一角度值**且此角度值为`0deg`或者`360deg`**时，如果线性渐变表达式不带上webkit的私有前缀`-webkit`的话，会得到不同的效果。如下图，

![](http://7xkwt1.com1.z0.glb.clouddn.com/CSS3渐变-005.png)

此时得到的两种效果其渐变方向却是不一样的。

至于原因，个人猜测可能是因为带上`-webkit-`私有前缀时和W3C规定的两种实现细节是不一样的。

#### 示例3

最后，我们可以在**渐变起点**和**渐变终点**之间插入任意多个色标，来达到多重颜色渐变的目的。比如，

```html
<style>
.gradient {
    width: 400px;
    height: 300px;
    font-size: 10px;
    margin-right: 15px;
    float: left;
    border: 1px solid;
    text-align: center;
    line-height: 80px;
}
.div1 {
    background-image: -webkit-linear-gradient(pink, blue, gold, green);
}
</style>
<div class=“gradient div1”>
    -webkit-linear-gradient(pink, blue, gold, green);
</div>
```

其效果如下，

![](http://7xkwt1.com1.z0.glb.clouddn.com/CSS3渐变-006.png)


# 重复线性渐变

我们可以使用`repeating-linear-gradient`来达到**重复线性渐变**的目的。所谓的重复线性渐变，就是沿着线性渐变线的两个方向上无线延伸重复。在这里需要注意一点就是，我们在设置渐变色标时应该同时给出其偏移量，否则无法实现重复渐变的效果。

让我们来看个简单的例子，

```css
.div1 {
    background-image: -webkit-repeating-linear-gradient(top left, pink 50px, green 100px);
}
```

其效果如下，

![](http://7xkwt1.com1.z0.glb.clouddn.com/CSS3渐变-013.png)



# 径向渐变

## 定义与语法

什么叫径向渐变？

不同于线性渐变，所谓的径向渐变其实就是圆形或者椭圆形渐变，其颜色渐变的轨迹不再是沿着一条直线，而是从一个点开始，向所有方向进行辐射。相对线性渐变要稍微复杂一些。

W3C规定的径向渐变标准语法如下，

```
background-image: radial-gradient(center, shape size, start-color, ..., last-color);
```

所以，`radial-gradient`将包含如下几个参数，

- 第一个参数为**渐变起点**
- 第二个参数为**渐变形状**和**渐变大小**
- 第三个参数为**渐变起点色标**
- 第四个参数为**渐变终点色标**

我们可以在**渐变起点色标**和**渐变终点色标**之间插入任意多个渐变中间色标。

### 渐变起点

所谓**渐变起点**即是径向渐变的的开始坐标。此参数可以使用常用的位置关键词，比如top、right、bottom、left、center，或者是*上右下左*的两两相邻组合。除此之外还可以赋值为类似`background-position`属性的值，即使用长度坐标或者百分比坐标。

**此参数的默认值为`center`**，且可以省略。

### 渐变形状

`shape`定义了渐变形状。径向渐变只有两种渐变形状，一种是椭圆（`ellipse`）一种是圆形（`circle`）。其实圆形径向渐变是一种特殊的椭圆径向渐变，因为此时径向渐变的**主半径**（major-radius）和**次半径**（minor-radius）是相同的。

**此参数的默认值为`ellipse`**，且可以省略。

### 渐变大小

`size`定义了渐变大小。其实此参数的本质是设置径向渐变的**主渐变半径**和**次渐变半径**。如下图，

![](http://7xkwt1.com1.z0.glb.clouddn.com/CSS3渐变-007.png)

其中水平方向是**主渐变半径**，垂直方向是**次渐变半径**。当主次半径一致时既是圆形，不一致时既为椭圆。

与线性渐变中的*长度值*参数相似，我们可以为渐变大小`size`设置一个css允许的度量单位，如px、em等，也可以使用百分比（％）。除此之外，我们还可以设置一些特殊的关键词，如下

- `closest-side`，指定径向渐变的半径长度为**从圆心到离圆心最近的边**
- `closest-corner`，指定径向渐变的半径长度为**从圆心到离圆心最近的角**
- `farthest-side`，指定径向渐变的半径长度为**从圆心到离圆心最远的边**
- `farthest-corner`，指定径向渐变的半径长度为**从圆心到离圆心最远的角**

当然，我们可以对`size`不作任何自定义设置，此时的默认值为`farthest-corner`。各个关键词的效果如下图，

![](http://7xkwt1.com1.z0.glb.clouddn.com/CSS3渐变-008.png)

**⚠️注意1**：如果我们将**渐变起点**、**渐变形状**及**渐变大小**这几个参数都省略，既都采用默认值，此时径向渐变的最终形状将根据容器的宽高来自动计算确定。

**⚠️注意2**：需要特别注意的是，**渐变形状**和**渐变大小**（即`shape`和`size`）的本质其实都是设置径向渐变的最终形状的，所以在某些时候这两个参数可能不能同时设置。

比如有如下的css代码，

```css
background-image: -webkit-radial-gradient(center, circle 50px 100px, pink, green);
```

那么浏览器将不知道如何渲染，因为你既设置了关键词`circle`，但是又同时设置了主半径和次半径，而且主次半径还不相同。我们知道主次半径不一致的结果将会呈现出一个椭圆，但是之前又明确设置了`circle`，此处将会有冲突。


### 渐变色标

径向渐变的色标跟线性渐变的色标类似。也有两个关键的渐变色标，一个是**渐变起点色标**，一个是**渐变终点色标**，且这两个色标都可以设置一个长度值（或者百分比）。

- 当为**渐变起点色标**设置长度（或者百分比）时，渐变起点将会发生偏移，偏移点为真正开始渐变的点。
- 当为**渐变终点色标**设置长度（或者百分比）时，渐变终点将会发生偏移，偏移点为真正终止渐变的点。
- 可以在**渐变起点色标**和**渐变终点色标**之间插入任意多个色标，来达到多色径向渐变的目的。

## 用法

### 基本用法

下面有一段示例，让我们一起来体验一下（为了方便，css部分的代码仅给出了webkit内核的声明，下同）

```html
<style>
.gradient {
    width: 400px;
    height: 400px;
    font-size: 10px;
    margin-right: 15px;
    float: left;
    border: 1px solid;
    text-align: center;
    line-height: 80px;
}
.div1 {
    background-image: -webkit-radial-gradient(pink, green);
}
.div2 {
    background-image: -webkit-radial-gradient(pink, green);
}
</style>

<div class=“gradient div1”>width: 400px; height: 400px;</div>
<div class=“gradient div2”>width: 400px; height: 200px;</div>
```

其效果如下图，

![](http://7xkwt1.com1.z0.glb.clouddn.com/CSS3渐变-009.png)

这是径向渐变最基本的使用方法，CSS代码中我们仅设置了**渐变色标起点**和**渐变色标终点**，其他的参数都采用默认值。从图中可以看出，

- 渐变起点为容器的中心，渐变方向为以起点为中心向四周辐射。
- 渐变颜色从内到外由粉色到绿色平滑过渡。
- 此时径向渐变的最终形状将由容器的大小确定。前一个容器的宽高一致，渐变形状为圆形，后一个容器的宽高不一样，渐变的形状为椭圆形。


### 更多用法

上面我们已经看到了径向渐变的基本用法，下面我们针对不同的参数分别做一些示例。

#### 示例1

首先我们来改变**渐变起点**，代码如下（为了简单这里仅给出了渐变相关的css代码）

```css
.div1 {
    background-image: -webkit-radial-gradient(center, pink, green);
}
.div2 {
    background-image: -webkit-radial-gradient(top, pink, green);
}
.div3 {
    background-image: -webkit-radial-gradient(top left, pink, green);
}
.div4 {
    background-image: -webkit-radial-gradient(33% 33%, pink, green);
}
```

效果如下，

![](http://7xkwt1.com1.z0.glb.clouddn.com/CSS3渐变-010.png)

从效果图中，我们可以看出，

- 渐变起点是可以自定义的，且可以设置为关键字、长度值、百分比。
- 在没有自定义渐变形状参数的情况下，最终的渐变形状受**渐变起点**和**容器大小**两个因素影响。

#### 示例2

其次，我们来改变径向渐变的形状或者大小参数，示例代码如下，

```css
.div1 {
    background-image: -webkit-radial-gradient(pink, green);
}
.div2 {
    background-image: -webkit-radial-gradient(center, circle, pink, green);
}
.div3 {
    background-image: -webkit-radial-gradient(center, 100px 100px, pink, green);
}
.div4 {
    background-image: -webkit-radial-gradient(center, 20% 40%, pink, green);
}
```

其效果如下，

![](http://7xkwt1.com1.z0.glb.clouddn.com/CSS3渐变-011.png)

从上面的效果图中，我们可以看出，

- **渐变起点**、**渐变形状**、**渐变大小**这几个参数都可以省略。他们的默认值分别为：`center`，`ellipse`，`auto`。其中渐变大小在未设置的情况下，将根据渐变起点和容器的大小自动计算。
- **渐变形状**的可选值有`circle`和`ellipse`。
- **渐变大小**可以设置为长度单位或者百分比。
- 当**渐变大小**手动设置为长度单位或者百分比时，**必须指定**渐变起点，否则浏览器会将自定义值作为渐变起点来渲染。
- **渐变大小**还可以使用默认的关键词（`closest-side`，`closest-corner`，`farthest-side`，`farthest-corner`）。此处效果图中未作示例，可参阅[渐变大小](http://blog.gejiawen.com/2015/04/23/css3-gradient-guide/#渐变大小)。

#### 示例3

最后，类似线性渐变，我们在**渐变起点色标**和**渐变终点色标**中可以插入任意多个**渐变色标**来得到多重径向渐变的目的。示例如下，

```css
.div1 {
    background-image: -webkit-radial-gradient(pink, blue, gold, green);
}
.div2 {
    background-image: -webkit-radial-gradient(pink 10%, blue 45%, gold 65%, green 80%);
}
```

其效果如下，

![](http://7xkwt1.com1.z0.glb.clouddn.com/CSS3渐变-012.png)

左边的效果未自定义任何色标偏移，右侧自定义了色标的偏移量。从图中我们可以看出，

- 在第一个和最后一个色标中间可以插入任意多个渐变色标。
- 对每一个色标都可以设置一个长度或者百分比，对渐变起点色标（第一个色标）和其他渐变色标（除了第一个色标之外）进行偏移。
- **渐变起点色标**和**其他色标**的偏移效果是不一致的。
- 对**渐变起点色标**进行偏移意味着**偏移点才是真正的渐变起点**。如上图中的P1处，P1的内部有部分容器（10%大小）是没有渐变的。
- 对**其他色标**进行偏移意味着**偏移点才是真正的渐变终点**。如上图中P2，P3，P4处。这三处位置在这里有两层含义，
    - 第一层含义是，**此处是当前色标的渐变终点**。比如P2位置处，它是blue颜色的渐变终点，理论上此处的颜色值取出来应该是`#00F`。
    - 第二层含义是，**此处时当前色标与下一个渐变色标的分割线**。比如P3处，它是gold颜色和green颜色的分割线。（其实P2位置也是一个分割线）
- 如果对**渐变终点色标**设置偏移，那么渐变终点将会提前终止，比如P4处。


# 重复径向渐变

我们可以使用`repeating-radial-gradient`来达到**重复径向渐变**的目的。与**重复线性渐变**一样，我们在设置渐变色标时需要同时给定偏移量，否则无法实现重复渐变的效果。

让我们来看个简单的例子，

```css
.div1 {
    background-image: -webkit-repeating-radial-gradient(top left, pink 50px, green 100px);
}
```

其效果如下，

![](http://7xkwt1.com1.z0.glb.clouddn.com/CSS3渐变-014.png)

# 参考列表

- [MDN:linear-gradient()](https://developer.mozilla.org/en-US/docs/Web/CSS/linear-gradient)
- [MDN:radial-gradient()](https://developer.mozilla.org/en-US/docs/Web/CSS/radial-gradient)
- [css3-gradients](http://www.w3cschool.cc/css3/css3-gradients.html)
- [css3-linear-gradient](http://www.w3cplus.com/css3/new-css3-linear-gradient.html)
- [css3-radial-gradient](http://www.w3cplus.com/css3/new-css3-radial-gradient.html)
- [深入理解CSS3 gradient斜向线性渐变](http://www.zhangxinxu.com/wordpress/2013/09/%E6%B7%B1%E5%85%A5%E7%90%86%E8%A7%A3css3-gradient%E6%96%9C%E5%90%91%E7%BA%BF%E6%80%A7%E6%B8%90%E5%8F%98/)



