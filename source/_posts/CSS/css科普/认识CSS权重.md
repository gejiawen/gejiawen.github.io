postid: "css-weight-guide"
title: 认识CSS权重
date: 2015-03-16 18:07:51
categories: [CSS]
tags: [CSS拾遗系列, css, css科普]
---

[CSS权重](http://www.w3.org/TR/CSS2/cascade.html#specificity)是在使用CSS进行布局时经常会遇到的一个问题，我们针对某一些元素应用一些样式，然后由于一些需求，使用更特殊的样式来覆盖以前的规则。所以，简单来说CSS权重就是确定元素上的样式应用规则的。

# 什么是CSS权重

一般来说，我们会有两条规则来确定CSS权重，

1. **从CSS代码的所处位置来看CSS权重优先级**: 内嵌样式 > 内部样式表 > 外部样式表。
2. **从样式选择器来看CSS权重优先级**： `!important` > 内嵌样式 > ID选择器（`#id`） > class选择器（`.class`） > 标签、伪类、属性选择器 > 伪元素 > 通配符（`*`） > 继承。

根据W3C的规范，CSS权重分为四个级别，格式如下，

**`a,b,c,d`**

- `a`：使用了内联样式，或者使用了`!important`
- `b`：形如`#id`的id选择器
- `c`：包含形如`.class`的类选择器、形如`xx[xx=xx]`的属性选择器、以及形如`xx:hover`的伪类
- `d`：包含标签选择器、伪元素选择器

各个权重位的高低关系是这样的，**a > b > c > d**。

针对一个css样式表达式，每满足上述的一个规则，则将对应的权重位增1（原始所有的权重位都默认为0）。

# 权重的计算

不过上述W3C的[文章](http://www.w3.org/TR/CSS2/cascade.html#specificity)中并没有对下列的情况做出说明，**如果不同的样式表达式对同一个元素设置了不同的样式，那么不同的样式表达式各自的权重是如何计算的？**

针对CSS权重，我们一般也会有两条规则来确定到底应用哪个样式，

1. **权重高者将会被应用**
2. **权重相同，则后定义的样式将会被应用**

出自国外的一篇文章[《CSS: Specificity Wars》](http://www.stuffandnonsense.co.uk/archives/css_specificity_wars.html)，现在有一种比较流行且容易理解的权重计算方案是这样的，

> 将各个权重位从高到低抽象成彼此10进制的进位，即`1000,100,10,1`，那么CSS样式表达式最终的权重为：`a`权重位上的数字x1000 + `b`权重位上的数字x100 + `c`权重位上的数字x10 + `d`权重位上的数字x1

如下图，

![](http://7xkwt1.com1.z0.glb.clouddn.com/认识CSS权重-001.png)

总结起来就是这样的，

![](http://7xkwt1.com1.z0.glb.clouddn.com/认识CSS权重-002.png)

举个例子，比如下面的代码，

```css
body #main .report h1 {
    color: red
}
```

其权重为分别是: **0,1,1,2**。（有一个id选择器，1个类选择器，2个标签选择器），所以最终的权重为**0x1000 + 1x100 + 1x10 + 2x1 = 112**。

## 验证demo

下面我们看个稍微复杂一点例子，来验证一下这种权重计算方法是否是没有问题。示例如下，

```css
#content div#main-content h2 { /*202*/
    color:red;
}
#content #main-content>h2 { /*201*/
    color:blue
}
body #content div[id="main-content"] h2 { /*113*/
    color:green;
}
#main-content div.paragraph h2 { /*112*/
    color:orange;
}
#main-content [class="paragraph"] h2 { /*111*/
    color:yellow;
}
div#main-content div.paragraph h2.first { /*123*/
    color:pink;
}
```

这里我已经在css代码注释中给出各个样式表达式的权重了，各位看官可以自己验证一下。

```html
<div id="content">
    <div id="main-content">
        <h2>CSS简介</h2>
        <p>CSS（Cascading Style Sheet，可译为“层叠样式表”或“级联样式表”）是一组格式设置规则，用于控制Web页面的外观。</p>
        <div class="paragraph">
            <h2 class="first">使用CSS布局的优点</h2>
            <p>1、表现和内容相分离 2、提高页面浏览速度 3、易于维护和改版 4、使用CSS布局更符合现在的W3C标准.</p>
        </div>
    </div>
</div>
```

那么问题来了。html代码中的两个`h2`前提将会渲染成什么颜色呢？

如果你已经知道答案了，可以到[这里](http://runjs.cn/detail/s2szltcq)看这个demo的在线预览，验证下自己的结论。

# 额外的问题

个人觉得理解CSS的权重特性是非常有用处的，比如可以压缩样式表达式的长度等。

这里，可能会有人对CSS权重的进制式有点疑问。

比如某个css表达式A最终权重是**0,1,0,0**，同时还有个css表达式B的最终权重**0,0,11,0**，那么按照进制式的算法，表达式A和B的权重分为别**100**和**110**。然后我们就可以得出结论，说因为B的权重因为比A大，所以元素将会渲染成B的样式。

其实这是**不正确**的。

我们说**进制式**的权重计算方案只是一种抽象，适用于大部分情况。但是也偶有例外，比如上面提到的这个例子。

W3C官方给出的权重方案中规定，

- 依次比较a,b,c,d，同位相同则再比较下一个。只要得到某一位较大者即停止继续往下比较。
- 若a,b,c,d都相同，则应用最后定义的样式

所以，权重并不是十进制的，而是**不进制**的。

# 参考列表

- [http://www.w3.org/TR/CSS2/cascade.html#specificity](http://www.w3.org/TR/CSS2/cascade.html#specificity)
- [CSS选择器的权重详解](http://www.cnblogs.com/rubylouvre/archive/2010/03/17/1687786.html)
- [CSS: Specificity Wars](http://www.stuffandnonsense.co.uk/archives/css_specificity_wars.html)
- [CSS选择器的权重与优先规则](http://www.nowamagic.net/csszone/css_SeletorPriorityRules.php)
- [你应该知道的一些事情——CSS权重](http://www.w3cplus.com/css/css-specificity-things-you-should-know.html)
- [重新认识CSS的权重](http://www.cssforest.org/blog/index.php?id=185)



