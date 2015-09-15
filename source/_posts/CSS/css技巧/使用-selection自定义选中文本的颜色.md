postid: "usage-for-selection-pseudo-class"
title: "使用::selection自定义选中文本的颜色"
date: 2015-04-14 01:42:31
categories: [CSS]
tags: [CSS拾遗系列, css, css技巧]
---

# ::selection

有的人在浏览网页时，喜欢一边选中文本一边阅读（博主就是这类强迫症患者）。在浏览某些个性鲜明的网站时，当我们划选文本时突然发现跟平时不太一样，比如下图，

![](http://7xkwt1.com1.z0.glb.clouddn.com/使用-selection自定义选中文本的颜色-001.png)

在windows环境下，正常的文本选中应该是深蓝色背景白色文本的样式。那么这个特殊的文本选中样式是如何做到的呢？

原来css中有这样的一个伪类`::selection`用于自定义文本选中时的样式设置。

`::selection`的语法如下，

```css
E::selection {
    background-color: xxx;
    color: xxx
}
```

让我们来看个例子，

```html
<style>
div::selection {
    background-color: red;
    color: #fff;
}
</style>
<div>
    blablabla......
</div>
```

其效果如下，

![](http://7xkwt1.com1.z0.glb.clouddn.com/使用-selection自定义选中文本的颜色-002.png)

这里有个[demo](http://runjs.cn/detail/yo70adwk)，感兴趣的看官可以点开看看。

关于这个伪类的文档貌似在W3C的官方文档中并没有明确的说明，不过在w3school中有[相关的说明](http://www.w3school.com.cn/cssref/selector_selection.asp)，如下

![](http://7xkwt1.com1.z0.glb.clouddn.com/使用-selection自定义选中文本的颜色-003.png)

经过实验，我们知道`::selection`伪类只能设置`color`、`background`、`cursor`、`outline`这几个属性。如果想要给选中文本设置`font-size`等属性，很可惜目前还不行。

# 参考列表

- [CSS3 ::selection 选择器](http://www.w3school.com.cn/cssref/selector_selection.asp)
- [CSS ::Selection](http://www.w3cplus.com/content/css-selection)



End! All rights reserved `@gejiawen`.


