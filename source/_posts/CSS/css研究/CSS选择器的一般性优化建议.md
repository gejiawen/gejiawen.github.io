postid: "suggestion-for-css-selectors"
title: "CSS选择器的一般性优化建议"
date: 2015-04-13 21:56:00
categories: [CSS]
tags: [CSS拾遗系列, css研究]

---


如何对选择器进行优化并不是一个那么新鲜的话题，但是个人觉得，css作为前端开发人员一项必备的专业技能，有必要深入了解下css在页面渲染的一些关键点，并以此为css的优化提供依据。我们这篇文章将讨论选择器优化的一般性建议。

# 浏览器如何应用选择器

在给出css选择器优化建议之前，我们有必要弄清楚这样一个问题，那就是**浏览器如何识别选择器并对相应的html元素进行样式渲染**？

浏览器对CSS的匹配原理是这样的，**浏览器读取选择器的顺序是由右到左进行**。

比如，

```css
div#divBox p span.red {
    color:red;
}
```

浏览器的匹配查找顺序如下，

1. 查找页面中所有`class=red`的`span`元素
2. 查找**1.**结果的父元素中是否有`p`元素
3. 查找**2.**结果的父元素中是否有`id=divBox`的`div`元素

如果以上3步都能找到相应的结果，那么则会给匹配的最终结果应用相关的css样式。


# css选择器的效率

从上面的结论和示例中，我们知道**浏览器是从右到左逐个匹配css表达式的**。那么为什么浏览器会采用这种**从右至左**的方法来匹配css表达式呢？其好处有两个，第一是为了尽早的过滤掉无关的样式规则第二是为了尽快的匹配到目标元素。很显然**表达式最右的样式规则**以及**整个css表达式的长度**将决定整个匹配过程的复杂度。这个最右部分的css表达式，我们称之为keyselector（**关键选择器**），它将决定css选择器的效率高低。

[这篇文章](http://csswizardry.com/2011/09/writing-efficient-css-selectors/)中给出了常规的css选择器的效率，

1. id选择器，比如`#header`
2. 类选择器，比如`.main`
3. 类型选择器，比如`div`
4. 兄弟选择器，比如`div + p`
5. 子元素选择器，比如`div > p`
6. 包含选择器，比如`div span`
7. 通配选择器，比如`div *`
8. 属性选择器，比如`input[type="text"]`
9. 伪类选择器，比如`a:hover,ul:nth-child(2n)`

可见id选择器的效率最高，而伪类选择器的效率最低。[CSS3提供的各种伪类选择器](http://blog.gejiawen.com/2015/04/09/first-to-css3-selectors/)虽然用起来很方便，不过其效率反而是最低的。

根据上面的理论，我们会得出下面的结论，`div #user`会比`#user div`的效率要高。因为前一个选择器表达式的关键选择器`#user`是一个id选择器，后一个选择器表达式的关键选择器`div`是一个类型选择器，而id选择器的效率要比类型选择器的效率要高。换句话说，前者经过关键选择器的匹配之后得到的集合要比后者要小，在一个更小集合上再次做匹配筛选，明显要更加快一点。

# css的优化建议

一般来说，我们书写css时应该遵循下面几点，

1. 规范css的命名和属性书写，能够体现出css代码的整齐性和条理性
2. 应用css优先级（[css权重](http://blog.gejiawen.com/2015/03/16/css-weight-guide/)）的知识避免冗余的兄弟选择器、子选择器、包含选择器，写出精简、合理的css表达式
3. 不到不得以勿用各种css hack
4. 合理使用css的绝对定位布局（绝对定位往往不太利于后续的维护和变更）
5. 合理使用css的继承
6. 尽量使用简写风格的css样式，将相似属性写在一起（主要是为了缩减css文件字节，加快浏览器渲染）
7. 限制css的嵌套层级，一般来说4层之内都是可以接受的
8. 模块化css样式（可以模仿业内流行css框架的做法，比如bootstrap）



# 参考列表

- [CSS选择器的优化](http://www.w3cplus.com/css/css-selector-performance)
- [Writing efficient CSS selectors](http://csswizardry.com/2011/09/writing-efficient-css-selectors/)
- [css原理及其优化](http://www.cnblogs.com/web-ed2/archive/2011/08/03/2126748.html)


