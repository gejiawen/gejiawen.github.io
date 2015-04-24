title: CSS拾遗系列
date: 2014-12-16 14:38:32
categories: [系列]
tags: [目录]

---

本文是一个随笔式的学习笔记，主要记录与css相关的一系列内容。

# 为什么会有这个系列

前段时间在讨论[产品](http://ce.baidu.com)的12月份迭代内容时，产品给出的需求是：要求首页改版，并且新版本的首页要求酷炫一点，特出产品的特点以及功能。最后要的要求是，使用前端技术做出一个比较连续的动画。本来我初步的设想是使用js来控制dom元素达到动画的效果，但是经过一些尝试之后，发现实在是太复杂，因为不同动画阶段之间的时序和dom元素之间的协调性不太好控制。后来转向了使用css3来实现动画效果然后再配合js做一些时序工作，发现效果还不错。

前面说了这么多，其实都是序曲。因为正式因为这件事让我意识到，我应该写一个系列博客来容纳一些关于CSS相关的东西。其实CSS，特别是CSS3，是一个很有搞头的东西，平时在工作中，我也遇到过各种非常巧妙的CSS的小技巧，当然也有遇到各种坑。我觉得把这些记录下来应该是很有意义的吧。

除了上面的理由之外，还有一个原因就是，我本人的CSS能力相比JavaScript而言是较弱的，因为其实我是一个半路出家的FE，之前对这些也没有系统的学习过，很多时候都是现学现卖。所以这个系列博客的另一个作用就是我学习CSS相关知识的一个沉淀。

本系列的文章将会包含如下的几个内容方向，

- 常用的或者不常用的CSS布局技巧
- 各种CSS兼容技巧
- 涉及CSS3的各种酷炫效果研究

可能以后还会增加更多的内容吧。:)

# 内容简要

本系列的文章将会按照内容分为几大类，不管是哪一类都是笔者学习CSS的沉淀。

# 目录

## 科普

[（1）CSS盒模型科普](http://gejiawen.github.io/2014/11/18/CSS/CSS%E7%9B%92%E6%A8%A1%E5%9E%8B%E7%A7%91%E6%99%AE/)
[（2）浅谈CSS中的伪元素和伪类](http://gejiawen.github.io/2014/11/20/CSS/CSS%E4%B8%AD%E7%9A%84%E4%BC%AA%E5%85%83%E7%B4%A0%E5%92%8C%E4%BC%AA%E7%B1%BB/)
[（3）CSS的长度单位参考](http://gejiawen.github.io/2015/03/03/CSS/CSS%E7%9A%84%E9%95%BF%E5%BA%A6%E5%8D%95%E4%BD%8D%E5%8F%82%E8%80%83/)
[（4）CSS布局模型](http://gejiawen.github.io/2015/03/04/CSS/CSS%E5%B8%83%E5%B1%80%E6%A8%A1%E5%9E%8B/)
[（5）CSS控制本文换行：word-wrap与word-break](http://gejiawen.github.io/2015/03/09/CSS/CSS%E6%8E%A7%E5%88%B6%E6%9C%AC%E6%96%87%E6%8D%A2%E8%A1%8C%EF%BC%9Aword-wrap%E4%B8%8Eword-break/)
[（6）认识CSS权重](http://gejiawen.github.io//2015/03/16/CSS/%E8%AE%A4%E8%AF%86CSS%E6%9D%83%E9%87%8D/)


## 技巧

[（1）CSS如何让页脚固定在页面底部](http://gejiawen.github.io/2014/12/16/CSS/CSS%E5%A6%82%E4%BD%95%E8%AE%A9%E9%A1%B5%E8%84%9A%E5%9B%BA%E5%AE%9A%E5%9C%A8%E9%A1%B5%E9%9D%A2%E5%BA%95%E9%83%A8/)
[（2）CSS如何使元素水平垂直定位](http://gejiawen.github.io/2014/12/21/CSS/CSS%E5%A6%82%E4%BD%95%E4%BD%BF%E5%85%83%E7%B4%A0%E6%B0%B4%E5%B9%B3%E5%9E%82%E7%9B%B4%E5%AE%9A%E4%BD%8D/)
[（3）CSS常用布局之宽高自适应](http://gejiawen.github.io/2015/03/12/CSS/CSS%E5%B8%B8%E7%94%A8%E5%B8%83%E5%B1%80%E4%B9%8B%E5%AE%BD%E9%AB%98%E8%87%AA%E9%80%82%E5%BA%94/)
[（4）CSS常用布局之各种元素的水平垂直居中](http://gejiawen.github.io/2015/03/13/CSS/CSS%E5%B8%B8%E7%94%A8%E5%B8%83%E5%B1%80%E4%B9%8B%E5%90%84%E7%A7%8D%E5%85%83%E7%B4%A0%E7%9A%84%E6%B0%B4%E5%B9%B3%E5%9E%82%E7%9B%B4%E5%B1%85%E4%B8%AD/)
[（5）使用::selection自定义选中文本的颜色](http://gejiawen.github.io/2015/04/14/CSS/%E4%BD%BF%E7%94%A8-selection%E8%87%AA%E5%AE%9A%E4%B9%89%E9%80%89%E4%B8%AD%E6%96%87%E6%9C%AC%E7%9A%84%E9%A2%9C%E8%89%B2/)


## 研究

[（1）WebFont与页面ICON图标研究](http://gejiawen.github.io/2015/03/04/CSS/WebFont%E4%B8%8E%E9%A1%B5%E9%9D%A2ICON%E5%9B%BE%E6%A0%87%E7%A0%94%E7%A9%B6/)
（2）CSS动画与@keyframes研究
[（3）CSS选择器的一般性优化建议]()


## CSS3

**注意**：由于CSS3中的部分内容尚未完全定稿，所以CSS3部分的系列文章可能会随时更新。

[（1）CSS3新特性概述](http://gejiawen.github.io/2015/04/13/CSS/CSS3%E6%96%B0%E7%89%B9%E6%80%A7%E6%A6%82%E8%BF%B0/)
[（2）初识CSS3选择器](http://gejiawen.github.io/2015/04/09/CSS/%E5%88%9D%E8%AF%86CSS3%E9%80%89%E6%8B%A9%E5%99%A8/)
[（3）CSS3多列布局](http://gejiawen.github.io/2015/04/14/CSS/CSS3%E5%A4%9A%E5%88%97%E5%B8%83%E5%B1%80/)
（4）CSS3弹性盒模型
[（5）CSS3背景](http://gejiawen.github.io/2015/04/21/CSS/CSS3%E8%83%8C%E6%99%AF/)
[（6）CSS3渐变](http://gejiawen.github.io/2015/04/23/CSS/CSS3%E6%B8%90%E5%8F%98/)
（7）CSS3阴影及反射
[（8）CSS3圆角](http://gejiawen.github.io/2015/04/23/CSS/CSS3%E5%9C%86%E8%A7%92/)
（9）CSS3媒体查询


# 总结

TODO

End. All rights reserved `@gejiawen`.


