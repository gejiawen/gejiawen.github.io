title: CSS控制本文换行：word-wrap与word-break
date: 2015-03-09 12:43:52
categories: [CSS]
tags: [CSS拾遗系列, css科普]
---

在页面上对文本进行修饰的时候，特别是针对西文文本，往往会涉及到单词过长需要换行的问题（这里暂不讨论[CJK](http://baike.baidu.com/link?url=9xJ8r-baLr_w8ahUivDOKViqJ3HnjpRlkc1eyqrJ6CYlShT-QPkq2paR6op_f9nj48QrSdiCm66oXKPDHm6NO_)系的文本）。本篇文章将会讨论两个用于控制文本换行的CSS属性`word-wrap`和`word-break`。

# `word-wrap`

我们先来看看[mozilla](https://developer.mozilla.org/en-US/docs/Web/CSS/word-wrap)上如何定义`word-wrap`的。

> The `word-wrap` CSS property is used to specify whether or not the browser may break lines within words in order to prevent overflow (in other words, force wrapping) when an otherwise unbreakable string is too long to fit in its containing box.

我们来尝试翻译一下这段定义，

> `word-wrap`属性用于说明是否允许浏览器在单词内进行断句（换句话说，就是是否能够强制换行），这是为了防止当一个字符串太长而找不到它的自然断句点时产生溢出现象。

 下图是`word-wrap`属性的详细内容，

 ![](http://7xkwt1.com1.z0.glb.clouddn.com/CSS控制本文换行：word-wrap与word-break-001.png)

其实有点CSS基础的人看到这段定义，应该就能明白这个CSS属性是干什么用的了。不过为了更加明白的阐述，我们下面来看个例子。

下面的页面是没有设置任何CSS换行控制的表现，

 ![](http://7xkwt1.com1.z0.glb.clouddn.com/CSS控制本文换行：word-wrap与word-break-002.png)

从图中可以看出来，句子最后个这个单词`see...e`实在是太长了，导致第一行已经放不下了。虽然浏览器默认将这个超长的单词放在第二行了，但是无奈这个容器仍然还是不够宽，最终导致的结果就是这个单词超出了容器，即所谓的溢出。

这里，其实有一点被隐藏的要素，就是，现代浏览器已经够聪明了，它知道第一行的空间已经放不下这个长单词了，尝试另起一行，将这个单词放到新行上，但是新起的一行仍然不能容纳这个长单词，此时浏览器也无能为力了。

此时，就需要我们的`word-wrap`属性登场了。

```css
.container {
    word-wrap: break-word;
}
```

效果如下图，

 ![](http://7xkwt1.com1.z0.glb.clouddn.com/CSS控制本文换行：word-wrap与word-break-003.png)

现在ok了，这个超长的单词`see...e`已经被强制换行了，虽然有点影响阅读。

# `word-break`

说完了`word-wrap`，我们再来说说`word-break`，先来看看[mozilla](https://developer.mozilla.org/en-US/docs/Web/CSS/word-break)是如何定义这个属性的，

> The `word-break` CSS property is used to specify how (or if) to break lines within words.

对应的翻译如下，

> CSS的`word-break`属性用于说明如何进行单词内的断句。

下图是`word-break`属性的详细内容，

 ![](http://7xkwt1.com1.z0.glb.clouddn.com/CSS控制本文换行：word-wrap与word-break-004.png)

这里我们使用如下语法，

```css
.container {
    word-break: break-all
}
```

我们再来看看效果，

 ![](http://7xkwt1.com1.z0.glb.clouddn.com/CSS控制本文换行：word-wrap与word-break-005.png)

看到没有，原来第一行空出来的那么点空间，现在也被填充了字符。

所以说，`word-break:break-all`是一个非常暴力的属性，它会为了充分利于页面空间将无视任何限制，对任何长单词在任何字符处进行断句（有时间这将非常影响文本的阅读）。

事实上，**`word-wrap:break-word`**与**`word-break:break-all`**共同点是都能把长单词强行断句，不同点是`word-wrap:break-word`会首先起一个新行来放置长单词，新的行还是放不下这个长单词则会对长单词进行强制断句；而`word-break:break-all`则不会把长单词放在一个新行里，当这一行放不下的时候就直接强制断句了。

# 参考列表

- [mozilla:word-wrap](https://developer.mozilla.org/en-US/docs/Web/CSS/word-wrap)
- [mozilla:word-break](https://developer.mozilla.org/en-US/docs/Web/CSS/word-break)
- [w3school:word-wrap](http://www.w3school.com.cn/cssref/pr_word-wrap.asp)
- [w3school:word-break](http://www.w3school.com.cn/cssref/pr_word-break.asp)
- [你真的了解word-wrap和word-break的区别吗？](http://www.cnblogs.com/2050/archive/2012/08/10/2632256.html)

End. All rights reserved `@gejiawen`.


