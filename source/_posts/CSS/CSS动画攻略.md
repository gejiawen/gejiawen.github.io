title: "CSS动画攻略"
date: 2015-05-05 17:18:30
categories: [CSS]
tags: [CSS拾遗系列, CSS研究]

---

在网页设计中给适当的元素添加动画交互无疑是一个大幅度提升用户体验的措施。纵观整个Web开发的发展历史，从最早的原始阶段（都是静态页面），到如今这样一个非常讲究用户体验的时代，人们越来越在意页面的表现能力（用户体验），干巴巴的展现内容往往是没有什么竞争力的。

就前端开发这个层面来说，实现动画的途径有好几种。简单来说可能包括下面几种方式，

1. 使用flash
2. 使用javascript不停的改变dom元素（包括一些第三内库，比如jquery等）
3. 使用css修饰dom元素

在过去的几年内，人们大都在使用第一种或者第二种方法来实现页面动画。现在看来前两种方法的确很low（第一种方法过重，第二种方法过于繁琐）。

CSS3的`transform`（变换），`transition`（过渡），`animation`（动画）三个属性为**纯CSS实现动画**提供了有力的支持。本篇文章我们将会详细的阐述各个属性的一般性用法，其中将会穿插部分示例。

# Transform（变换）

CSS3新增了属性`transform`，意为**变换**。那么，什么叫做变换？简单的来说就是改变元素的大小、位置、形状等属性的操作。

**⚠️注意**：其实`transform`还可以分为**2D变换**和**3D变换**。前者表示在二维空间进行变换操作（即只有X轴和Y轴），而后者表示在三维空间进行变换操作（即除了二维的X轴和Y轴之外还有一个Z轴）。本篇文章仅对2D变换做阐述，3D变换的相关内容后面会专门作文。

W3C中规定`transform`包含如下几种操作类型（**这里所有的变换都是在2D变换的范畴内**），

- `translate` 位移
- `rotate` 旋转
- `scale` 缩放
- `skew` 倾斜
- `matrix` 变换矩阵

一般来说，`transform`的语法如下，

```
transform: none | <transform-function> [ <transform-function>]*
```

1. `none`为默认值，意为不作任何变换
2. `<transform-function>`即为上述提到的各种变换操作
3. 我们可以同时对一个元素进行多重变换操作，即可以赋值多个`<transform-function>`
4. 当进行多重变换操作时，不同的变换操作之间使用**空格**分割而不是采用一贯的`,`，切记。

## `translate-origin`属性

在介绍各种变换操作之前，我觉得很有必要先介绍一下`tranform-origin`属性。此属性不属于`transform`的操作类型，它和`transform`属于同一个level的css属性，它的作用是**用于设置变换操作的原点**。

其语法如下，

```
transform-origin: [<percentage> | <length> | left | center | right] [<percentage> | <length> | top | center | bottom]?
```
1. 默认值为`center center`（`50% 50%`）,默认变换原点为元素的中心点。
2. 此属性接收1-2组值，分别应用于X轴与Y轴。如果只设置第一组值，则第二组值将自动采用默认值。
3. 当参数值为`<percentage> | <length>`时，可以为负值。

**记住**，所有的二维变换操作都必须基于一个原点，默认的原点为元素的中心点。

此参数的浏览器兼容性如下，

![](img-01.png)

部分支持的浏览器需要我们在实际使用时添加各个浏览器的私有前缀。

下面我们来看一个例子见识一下改变`transform-origin`后的变换效果，

```css
.demo {
    transform-origin: 100% 0;
    transform: translate(50px, 50px);
}
```

效果如下，

![](img-00.png)

图中左侧的效果应用了`transform-origin: 100% 0;`导致其旋转变换原点转移到了右上角。可于右侧未设置变换原点的效果图进行对比体会其变化。

## `translate`

`translate`意为**位移**。它的一般性用法如下，

```css
.demo {
    transform: translate(50px, 50px);
}
```

上面的代码表示将`.demo`元素以其变换原点（元素中心点）为基准，各向右（X轴）和向下（Y轴）移动50px。其效果如下图，

![](img-02.png)

**注意**：从上图的效果中，我们可以看出`translate`其实是通过将元素的中心点移动到新的位置来使得整个元素达到位移目的的。

`translate(50px, 50px)`是位移操作最基本的用法，其还有如下几种变形，

- `translate(50px)`，省略第二个参数，此时只进行X轴位移，Y轴位移量默认为0。
- `translate(0, 50px)`，可以显式的声明X轴位移量为0，这样仅进行Y轴位移。
- `translateX(50px)`，等同于`translate(50px, 0)`，只进行X轴位移。
- `translateY(50px)`，等同于`translate(0, 50px)`，只进行Y轴位移。

## `rotate`

`rotate`意为**旋转**。它的一般性用法如下，

```css
.demo {
    transform: rotate(45deg);
}
```

上面代码表示将`.demo`元素以其变换原点（元素中心点）为基准，顺时针旋转45度。效果如下，

![](img-03.png)

- `rotate(45deg)`，接收一个参数，此参数表示旋转的度数。
- `rotate`的参数可以传入一个正值，亦可以传入一个负值（比如`-45deg`）。正值表示旋转方向为**顺时针**，负值表示旋转方向为**逆时针**。

## `scale`

`scale`意为**缩放**。它的一般性用法如下，

```css
.demo {
    transform: scale(1.5);
}
```

上面的代码表示将`.demo`元素以其变换原点（元素中心点）为基准，沿着X轴和Y轴**放大**1.5倍。效果如下，

![](img-04.png)

`scale`操作与`translate`很相似，`scale(1.5)`是其最基本的用法，除此之外它还有如下几种变形，

- `scale(1.5, 1.5)`，可以接收两个参数，分别表示X轴方向与Y轴方向的缩放。其中第二个参数可以省略。若省略第二个参数，则**第二个参数与第一个参数一致**。
- 传入的参数如果是<1的值则按比例缩小，若是>1的值则按比例放大。不可传入负值。
- 可以使用`scaleX()`或者`scaleY()`进行**仅对**X轴方向或者Y轴方向进行缩放。

## `skew`

`skew`意为**倾斜**。它的一般性用法如下，

```css
.demo {
    transform: skew(30deg, 30deg);
}
```

上面的代码表示将`.demo`元素以其变换原点（元素中心点）为基准，沿着X轴顺时针倾斜30度，沿着Y轴顺时针倾斜30度。效果如下，

![](img-05.png)

**注意**：`skew`在二维变换中是唯一一个可以使元素发生**形变**的操作。

- `skew`与之前的`translate`和`scale`也是类似的，接收2个参数。其中第二个参数可以省略，若省略第二个参数，则第二个参数将自动使用默认值`0deg`。
- 可以使用`skewX()`和`skewY()`进行**仅**X轴方向或者Y轴方向的倾斜。
- 参数可以设置为正值亦可以设置为负值。其中正值表示顺时针方向，负值表示逆时针方向。

## `matrix`

关于`matrix`的详细用法，我这里不想多作阐述。因为这种用法基本很少用到，而且还涉及到一些行列式变换的运算。就是说了我估计我也说不清楚。这里只要知道`matrix`是合并了所有的变化操作就行，如果实在有兴趣，可自行查阅相关文档。

其实，`transform`允许对同一元素同时进行多个变换操作，这样完全可以替代`matrix`的用法，而且可读性更佳。

## `transform`总结

看到这里，可能会一些看官有个疑问，**貌似`transform`并不能实现动画啊？**

没错！

仅仅使用`transform`是无法实现动画效果的。因为`transform`只是对元素进行生硬的变化，这个变化是瞬间的，所以并没有展现一种持续变化的效果。

一般地，我们可以将其与下面即将说道的`transition`搭配使用来得到各种动画效果。

不过，在适当的时候使用`transform`可以大大简化页面实现复杂度，比如有一个比较复杂的展示元素，我们现在需要一个与之水平对称的展示，那么此时只需要`rotate`即可。

*上面所有效果可以在这个[demo](http://runjs.cn/code/cgqbfrd7)中进行试验。*

# Transition（过渡）

W3C中[css3-transition](http://www.w3.org/TR/css3-transitions/)的文档对`transition`的定义是这样的，

> CSS Transitions allows property changes in CSS values to occur smoothly over a specified duration.

意思就是：**允许css的属性值在一定的时间区间内平滑地过渡。**这种效果可以在鼠标单击、获得焦点、被点击或对元素任何改变中触发，并圆滑地以动画效果改变CSS的属性值。

简单来说，`transition`的含义就是元素某一css属性的**过渡**。

下面让我们来详细看看`transition`这货到底是怎么用的。

`transition`的语法如下，

```
transition：[ transition-property ] || [ transition-duration ] || [ transition-timing-function ] || [ transition-delay ]
```

可以看出`transition`是一个复合属性，它包括下面四个子属性，

- `transition-property`，用于设置需要过渡的css属性
- `transition-duration`，用于设置过渡的持续时间
- `transition-timing-function`，用于设置过渡的速率类型（即过渡的动画类型）
- `transition-delay`，用于设置过渡的开始延时

此外`transition`允许使用`,`同时对一个元素进行多个属性的过渡，比如

```css
.demo {
    transition: width 1s ease-out, height 1s ease-in;
}
```

这样我们可以**对同一个元素的不同的属性应用不同的过渡策略**。

下面我们来分别对每个属性进行描述，其中可能会穿插一些示例。

## `transition-property`

用于设置需要过渡的css属性，语法如下，

```css
transition-property：all | none | <property>[ ,<property> ]*
```

- `none`，不指定过渡的css属性
- `all`，此为默认值，指定所有可进行过渡的css属性
- `<property>`，可自定义需要过渡的css属性

这里值得一提的是，**不是所有的css属性都是可以进行过渡的**。比如`display`属性，比如`background-image: url(...)`（当设置背景图片为一个gradient时反而是可以应用过渡的），还有一些没有明确大小的单位，比如`height: auto`，等等。

[这篇文章](http://oli.jp/2010/css-animatable-properties/)中有一份可过渡的css属性列表。列表不但指明了哪些css属性是可以过渡的，还指明了这些css属性的取值类型（意思就是，这些css属性必须满足这些取值条件才可以应用过渡效果）。

这里还有一个[demo](http://leaverou.github.io/animatable/)，展示了几乎所有的`transition`效果。

## `transition-duration`

用于设置过渡的持续时间，语法如下，

```
transition-duration：<time>[ ,<time> ]*
```

这个属性没什么可说的，就是用于设置过渡的持续时间。过渡时间越短则动画运行速度越快，过渡时间越长则动画运行越慢。

此属性的单位为秒（`s`），默认值为0。当设置为0时，则表示过渡是瞬间完成的。

## `transition-timing-function`

用于设置过渡的速率类型，语法如下，

```css
transition-timing-function：linear | ease | ease-in | ease-out | ease-in-out | step-start | step-end | steps(<integer>[, [ start | end ] ]?) | cubic-bezier
```

这个属性相比之前的两个属性要稍微复杂一点，它的设置非常多样化。

- 我们可以使用预设的几种类型：`linear`（线性过渡），`ease`（平滑过渡，**此为默认值**），`ease-in`（由慢到快），`ease-out`（由快到慢），`ease-in-out`（由慢到快再到慢）
- `cubic-bezier()`，自定义[贝塞尔曲线](http://baike.baidu.com/link?url=bMro0Qq1B_EJZneis-WAahVi3fU8HWsmcagdWiOz8KcOilhsLPpVhyAA6hJMPB3FN4kBEuKWI_zTLBk0flxyva)。其实预设的几种都有对应的贝塞尔曲线。
- `steps(<integer>[, [ start | end ] ]?)`，接受两个参数的步进函数。第一个参数必须为正整数，指定函数的步数。第二个参数取值可以是start或end，指定每一步的值发生变化的时间点。第二个参数是可选的，默认值为end。
- `step-start`，等同于`steps(1, start)`
- `step-end`，等同于`steps(1, end)`

我想这里唯一需要解释的就是`steps`相关的几种过渡速率类型。`steps()`的意思就是将整个过渡过程分为特定的步数，并规定了每一步过渡的执行时机。

比如，

```css
transition-property: all;
transition-duration: 2s;
transition-timing-function: steps(5, end);
```

这段代码的效果请参见这个[demo](http://runjs.cn/detail/inm5onf7)。

`steps(5, end)`的意思就是，

1. 将整个持续2s的过渡过程分为5个步骤。
2. 每一步的过渡效果将在每一步结束时发生（end）。

[Lea Verou](https://github.com/LeaVerou)的[http://cubic-bezier.com/](http://cubic-bezier.com/)可以可视化定制各种贝塞尔曲线。

[MDN](https://developer.mozilla.org/en-US/docs/Web/CSS/transition-timing-function)上有对每一种过渡速率类型的效果展示。

## `transition-delay`

用于设置过渡的开始延时，其语法与`transition-duration`是一样的。这里就不再赘述了。

不过有一点需要注意，当我们使用`transition`简写模式的时候，一般按照如下的格式来书写，

```css
.demo {
    transition: all 2s ease 1s;
}
```

除此之外，你可能还会看到别人这么写，

```css
.demo {
    transition: 2s 1s
}
```
仅指明了两个时间参数，其他的都使用默认参数。此时第一个时间参数为`transition-duration`，第二个时间参数为`transition-delay`。

除此之外，我们还可以利用`transition-delay`属性来达到一些酷炫的效果，比如

```css
.demo {
    transition: height 1s, width 1s 1s;
}
```

意思就是先对`height`属性进行过渡，延迟1s后对`width`属性进行过渡，而这个延迟的时间正好是前一个属性的过渡时间，从而整体看起来非常平滑。详情可参阅这个[demo](http://runjs.cn/detail/dewafhg9)。

## `transition`总结

`transition`简单明了，可快速对元素实现一些平滑过渡。不过它也有自己的适用场景和注意事项。

下面是一些使用`transition`的注意事项，

1. 由于一些浏览器支持原因，在使用`transition`时最好添加各个浏览器厂商的私有前缀，将W3C的语法放在最后。
2. 不是所有的CSS属性都支持`transition`。
3. `transition`需要明确知道开始状态和结束状态的具体数值，才能计算出中间状态。

`transition`的局限，

1. `transition`必须要外力推动，比如鼠标动作，或者js操作等等，否则无法触发过渡。
2. `transition`的过渡效果是一次性的，除非人为的反复触发。
3. `transition`只能接受两个边界状态（开始和结束），中间的过渡状态是由浏览器自动计算的。无法人为的指定中间状态。

为了解决上述问题，CSS Animation应用而生了。

# Animation（动画）

`animation`相比`transition`制作动画拥有更高的自定义性，可以定义每一帧的行为。

首先让我们来看一个模拟月食的[demo](http://runjs.cn/detail/f5keiqb4)来体验一下CSS动画的魅力。

哈哈，感觉如何，是不是有种很酷炫的感觉。下面让我们来详细看看`animation`的用法吧。

`animation`的语法如下，

```css
animation: [[ animation-name ] || [ animation-duration ] || [ animation-timing-function ] || [ animation-delay ] || [ animation-iteration-count ] || [ animation-direction ]] [ , [ animation-name ] || [ animation-duration ] || [ animation-timing-function ] || [ animation-delay ] || [ animation-iteration-count ] || [ animation-direction ]]*
```

与`transition`一样，我们可以对同一个元素同时应用多个`animation`，比如，

```css
@keyframes typing {
    from { width: 0; }
}
@keyframes blink-caret {
    50% { border-color: transparent; }
}
h1 {
    font: bold 200% Consolas, Monaco, monospace;
    border-right: .1em solid;
    width: 16.5em; /* fallback */
    width: 30ch; /* # of chars */
    margin: 2em 1em;
    white-space: nowrap;
    overflow: hidden;
    animation: typing 20s steps(30, end), blink-caret .5s step-end infinite alternate;
}
```

上述代码可以实现一个非常神奇的效果，它给`h1`元素同时应用了`typing`和`blink-caret`两种动画，具体的效果可参见[这里](http://dabblet.com/gist/1745856)。

`animation`可设置的属性还是挺多的，下面让我们一个个的看看每个属性的具体用法。

## `animation-name`

`animation-name`意为动画的名称。如果不设置`animation-name`，那么元素将没有任何的动画效果。

其语法如下，

```
animation-name：none | <identifier> [ , none | <identifier> ]*
```

- `none`，此为默认值，表示不引用任何动画。
- `<identifier>`，引用由`@keyframes`定义的动画名称。

说到这个`animation-name`属性，那就不得不说一下`@keyframes`的相关知识。

## `@keyframes`

`@keyframes`用于定义动画的各个状态，我们一般称之为关键帧。其写法非常自由。其一般的写法如下，

```css
@keyframes rainbow {
    0% { background: #c00 }
    50% { background: orange }
    100% { background: yellowgreen }
}
```

- rainbow即为动画关键帧的名字，被`animation`引用。
- 内部可以使用`0%`~`100%`自定义动画的各个状态。其中`0%`表示初始状态，`100%`表示结束状态。
- 还可以用`from`来代替`0%`表示初始状态，可以用`to`关键字来代替`100%`表示结束状态。
- 未明确的状态将由浏览器自动计算，且不同状态之间是平滑过渡的（究竟如何过渡，取决`animation-timing-function`属性）。
- 还可以将多个状态写在一起表示这几个状态是一样的，比如

```css
@keyframes moonline {
    0% {
        top:220px;
        left:30%;
        opacity:0;
    }
    30%, 50%, 80% {
        top:100px;
        left:50%;
        opacity:1;
    }
    100% {
        top:220px;
        left:80%;
        opacity:0;
    }
}
```

## `animation-duration`

`animation-duration`意为动画持续时间，跟之前的`transition-duration`的含义一致。

其语法如下，

```
animation-duration：<time> [ , <time> ]*
```

- 设置动画的持续时间，单位为秒（`s`），其默认值为0。

## `animation-timing-function`

`animation-timing-function`用于设置动画的速率类型。这个属性与`transition-timing-function`基本一致。

其语法如下，

```
transition-timing-function：linear | ease | ease-in | ease-out | ease-in-out | step-start | step-end | steps(<integer>[, [ start | end ] ]?) | cubic-bezier
```

具体的解释这里就不再赘述了，可参考[`transition-timing-function`](http://gejiawen.github.io/2015/05/05/CSS/CSS%E5%8A%A8%E7%94%BB%E6%94%BB%E7%95%A5/#transition-timing-function)。

## `animation-delay`

`animation-delay`用于设置动画开始的延时。此属性与`transition-delay`基本一致，请参考[`transition-delay`](http://gejiawen.github.io/2015/05/05/CSS/CSS%E5%8A%A8%E7%94%BB%E6%94%BB%E7%95%A5/#transition-delay)的相关说明。

## `animation-iteration-count`

此属性用于设置动画的循环次数。其语法如下，

```
animation-iteration-count：infinite | <number> [ , infinite | <number> ]*
```

- 此参数可以设置为一个`<integer>`，明确标示动画的循环次数。默认值为1，即动画只运行一次。
- 还可接收关键字`infinite`，表示动画将运行无限次。

## `animation-direction`

简单来说，此属性用于设置动画在循环运行过程中是否允许反向运动。这啥意思呢？

我们知道当动画循环运行时，一次运行周期为：从初始状态到结束状态的平滑过渡。一次运行周期结束后，元素再次下一个周期，即又从初始状态平滑过渡到结束状态。我们可以利用`animation-direction`属性改变动画运行偶数周期的行为。

此参数的语法如下，

```
animation-direction：normal | alternate | reverse | alternate-reverse
```

- `normal`，表示使用默认的行为，不作任何改变。
- `alternate`，允许动画在偶数周期期间，让元素从结束状态平滑过渡到初始状态。
- `reverse`，动画的所有运行周期中，让元素从结束状态平滑过渡到初始状态。
- `alternate-reverse`，动画的奇数周期内过渡方向与设置的方向相反，偶数周期内过渡方向与设置的方向相同。

这四个预定义值，一般比较常用的是`normal`和`alternate`。下面是一个例子，

```css
@keyframes rainbow {
    0% {
        background-color: yellow;
    }
    100% {
        background: blue;
    }
}
.demo:hover {
  animation: 1s rainbow 3 normal;
}
.demo2:hover {
  animation: 1s rainbow 3 alternate;
}
.demo3:hover {
  animation: 1s rainbow 3 reverse;
}
.demo4:hover {
  animation: 1s rainbow 3 alternate-reverse;
}
```

其效果如下，

![](img-06.png)

*本demo来自[CSS动画简介](http://www.ruanyifeng.com/blog/2014/02/css_transition_and_animation.html)。*

## `animation-play-state`

语法如下，

```
animation-play-state:running | paused [, running | paused]* 
```

- 此属性用于控制动画的运行状态。
- 有两个可选值，`running`（运动状态）和`paused`（暂停状态）。其中`running`为默认值。
- 当某个动画从暂停状态->运动状态时，它并不会重新从初始状态开始，而是在上次暂停的状态继续运行动画。

下面让我们来看一个例子。

```css
div {
	animation: spin 2s linear infinite;
	animation-play-state: paused;
}
div:hover {
    animation-play-state: running;
}
@keyframes spin {
    from {
        transform: rotate(0deg);
	}
	to {
	    transform: rotate(360deg); 
	}
}
```

具体的效果请参见这个[demo](http://runjs.cn/detail/catoma9r)。


## `animation-fill-mode`

此属性用于设置动画时间之外元素的状态。这里的**动画时间之外**可能包括*动画开始运行之前*、*动画结束运行之后*、*动画暂停运行时*。

其语法如下，

```
animation-fill-mode：none | forwards | backwards | both
```

- `none`，此为默认值，意为不设置动画之外的状态。
- `forwards`，设置状态为动画结束时的状态。
- `backwards`，设置状态为动画开始时的状态。
- `both`，设置状态为动画结束或开始的状态。当循环运行动画时，`both`会更具根据`animation-direction`轮流应用`forwards`和`backwards`规则。

详情可参考这个[demo](http://runjs.cn/detail/dzkabsra)。

## `animation`总结

其实`animation`可以说是`transition`的进阶版本。`transition`针对元素的某一个（或者某几个）特定的css属性进行过渡，而`animation`可以针对元素在过渡期间自定义任意一帧的状态。如果不考虑低版本浏览器的兼容性，使用css动画来提升用户体验是一种非常好得方式，

个人感觉，从某种意义上来说，设计师是决定用户体验的第一道门槛，CSS的相关内容都是实现。在实际的工作中，我们可以发挥自己的想象力，使用css动画的相关知识，组合各种效果。


# 参考列表

- [http://www.w3.org/TR/css3-2d-transforms](http://www.w3.org/TR/css3-2d-transforms/)
- [http://www.w3.org/TR/css3-3d-transforms](http://www.w3.org/TR/css3-3d-transforms/)
- [http://www.w3.org/TR/css3-transitions](http://www.w3.org/TR/css3-transitions/)
- [http://www.w3.org/TR/css3-animations](http://www.w3.org/TR/css3-animations/)
- [CSS动画简介](http://www.ruanyifeng.com/blog/2014/02/css_transition_and_animation.html)
- [CSS3 Transitions, Transforms和Animation使用简介与应用展示](http://www.zhangxinxu.com/wordpress/2010/11/css3-transitions-transforms%E5%92%8Canimation%E4%BD%BF%E7%94%A8%E7%AE%80%E4%BB%8B%E4%B8%8E%E5%BA%94%E7%94%A8%E5%B1%95%E7%A4%BA/)
- [css in the 4th dimension](http://lea.verou.me/css-4d/#intro)
- [CSS3 Transform](http://www.w3cplus.com/content/css3-transform)
- [CSS3 Transition](http://www.w3cplus.com/content/css3-transition)
- [CSS3 Animation](http://www.w3cplus.com/content/css3-animation)

End! All rights reserved `@gejiawen`.
