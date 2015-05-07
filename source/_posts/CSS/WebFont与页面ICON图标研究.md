title: WebFont与页面ICON图标研究
date: 2015-03-04 14:20:34
categories: [CSS]
tags: [CSS拾遗系列, CSS研究]

---

当你打开（绝大部分）网站，页面上将会有许多形形色色的小图标（icon），适当的icon的可以达到一图胜千言的目的，使网页的表现效果更佳。

关于页面icon的制作，比较传统的方法是，让设计师去设计一个个的小图片，然后网页程式员再将设计好的icon放到页面上适当的位置。不过，随着这些年Web前端技术的迅猛发展，现在有一种新的方案去制作页面icon，那就是webfont。

本文将较为详细的介绍页面icon及webfont的方方面面。

# 页面icon

## 什么是页面icon

如下图所示，

![](img-1-1.png)

[天猫商城](http://tmall.com)中左侧的导航栏中，每一个购物频道都有一个小小的图标，这些小图标就是页面icon的一种表现方式。

除此之外，页面icon还有多种表现形式，可能出现在页面的任何位置。

## 传统icon的制作

我们再来看看[百度](http://baidu.com)域下的资源文件，其中有一个[图片文件](https://ss1.bdstatic.com/5eN1bjq8AAUYm2zgoY3K/r/www/cache/static/protocol/https/global/img/icons_3bfb8e45.png)，如下图，

![](img-1-2.png)

从图中可以看出来，这个图片中有很多的小图标，这些小图标按照一定的顺序排列在一起。

上面这种方式就是我们所说的传统制作和使用icon的方法。

首先要求设计师设计好大小合适的小图标，然后将这些小图标按照一定的顺序和方式合并在一起（这种方式称为css sprite，或者图片精灵），然后网页程序员通过书写css代码来控制相应元素的`background-position`属性，以达到不同元素显示不同的小图标。

这种方式下，需要使用小图标的页面元素的css规则一般这样定制：一个表示图标类的`icon`样式，以及表达不同小图标的自定义类名，比如`icon-home`，`icon-user`。当然，具体css类名的命名规范是不定的。一般地，`icon`和`icon-*`的样式内容如下，

```css
.icon {
    background-image: url(...)
}
.icon-home {
    background-position: 0 0;
}
.icon-user {
    background-position: 0 10px;
}
```

这种制作和使用icon的方式现在仍然有许多企业和网页正在使用，也是一种比较常规的方式。这种方式在书写css代码需要有一定的耐心，要匹配好各个icon的`background-position`属性。

值得一提的是，这种方式有一个不可避免弊端，就是，可能页面的icon需要两种以上的尺寸或者icon要发生变更。前者一般会要求设计师产出多套的icon，因为直接对图片进行缩放在网页上的表现并不是很好；后者可能就要重写之前的css代码了，因为可能图片雪碧后的position也发生了变化。

# webfont与@font-face

## 什么是webfont

随着这些年Web前端技术的迅猛发展，web font技术逐渐成熟。那么什么是web font呢？

web font，又称之为**在线字体**或者**网络字体**，是CSS3中的一个模块，主要是把自定义的特殊字体嵌入到网页中。无需安装，无需下载，直接在线使用。

## @font-face语法

web font技术需要通过CSS的**`@font-face`**语句引入在线字体。所以这里我先说一下@font-face的相关内容。

@font-css是CSS3中的一个模块，通过它可以将自定义的字体嵌入到前端网页中。随着@font-face的出现，标识着我们在web开发的过程中可以使用除了web安全字体之外的自定义字体，使页面的展现更加多样化。

值得一提的是，@font-face这个CSS3模块早在IE4中就已经被支持了。有点意外。

我们先来看看@font-face的语法，

```css
@font-face {
    font-family: <your-webfont-name>;
    src: <source> <format> [, <source> <format>];
    [font-weight: <weight>;]
    [font-style: <style>;]
}
```

值得注意的有两点，一个是`font-family`属性，一个是`src`属性。前者是自定义webfont的名字，后者是引用字体的路径。其中`src`中`<format>`字段是用来标识字体格式帮助浏览器识别。常见字体格式及format如下图，

![](img-2-2-00.png)

format说明，

![](img-2-2-01.png)

而浏览器对各字体格式的支持如下，

![](img-2-2-02.png)

说了这么多的理论，下面让我们来一段具体的CSS代码，了解下这个@font-face到底是如何定义的。

```css
@font-face {
    font-family: 'icomoon';
    src:url('fonts/icomoon.eot?'); /* 兼容IE9以上 */
    src:url('fonts/icomoon.eot?#iefix') format('embedded-opentype'), /*兼容IE8以下*/
        url('fonts/icomoon.woff') format('woff'),
        url('fonts/icomoon.ttf') format('truetype'),
        url('fonts/icomoon.svg') format('svg');
    font-weight: normal;
    font-style: normal;
}
```

这样我们自定义的web font就成功了。然后就可以在页面中正常使用了。比如，

```css
div.title {
    font-family: 'icomoon'
}
```

如果不出任何的意外的话，你将会得到自定义的字体效果。

## 自定义字体

说到这里，如果大家自己动手实验一番的话，就会发现一个致命问题：**我去哪里获得这些自定义字体啊？**

目前有三种途径来获取这些字体，

- 去免费的网站下载字体
- 去收费的网站购买字体使用授权
- 有设计背景，自己设计字体

针对前两种方式没什么好说的，针对第三种方案，可能相关门槛就高了一点，需要一些设计背景。如果有兴趣，可以参阅[这篇文章](http://flowerboys.cn/font/article/fontsArticle/how-to-turn-your-icons-into-a-web-font.html)。

# 字体图标

## 使用现有的免费字体图标

有一个好用的[html5应用](https://icomoon.io/app)。通过此应用你可以选择IcoMoon提供的免费或者购买收费图标，然后生成字体。如下图所示，

![](img-3-1-00.png)

下载得到生成好的字体后，它会帮我们生成好css代码，如下，

```css
@font-face {
    font-family: 'icomoon';
    src:url('fonts/icomoon.eot?-rah7ee');
    src:url('fonts/icomoon.eot?#iefix-rah7ee') format('embedded-opentype'),
        url('fonts/icomoon.woff?-rah7ee') format('woff'),
        url('fonts/icomoon.ttf?-rah7ee') format('truetype'),
        url('fonts/icomoon.svg?-rah7ee#icomoon') format('svg');
    font-weight: normal;
    font-style: normal;
}
.icon {
    font-family: 'icomoon';
    speak: none;
    font-style: normal;
    font-weight: normal;
    font-variant: normal;
    text-transform: none;
    line-height: 1;
    /* Better Font Rendering =========== */
    -webkit-font-smoothing: antialiased;
    -moz-osx-font-smoothing: grayscale;
}
.icon-home3:before {
    content: "\e600";
}
.icon-office:before {
    content: "\e601";
}
.icon-newspaper:before {
    content: "\e602";
}
.icon-pencil:before {
    content: "\e603";
}
```

ok，事已至此，我们已经得到了需要的字体、css代码。现在我们可以在html页面使用图标字体来制作页面小icon了。

这里我们有两种方式来使用图标字体，一种是**html实体 + webfont**，一种是**css样式 + webfont**。

## html实体+webfont

还记得文首提到的天猫首页中的小icon图标么？现在我再来看看它的源代码，

```html
<li>
    <s class="menu-nav-icon fp-iconfont">&#x 3451;</s>
</li>
<li>
    <s class="menu-nav-icon fp-iconfont">&#x 3459;</s>
</li>
<li>
    <s class="menu-nav-icon fp-iconfont">&#x 346c;</s>
</li>
```

```css
.fp-iconfont {
    font-family: tm-fp-font !important;
}
```

这里我们只截取了一部分html代码，并去除了多余的样式因素。

从上述的代码中，我们看到`s`标签中包括了一些十六进制字符（以`&#x`开始，我这里强制添加了一个空格，不然实体就被浏览器解析了），这些奇怪的字符的作用就是配合自定义font-family进行页面渲染，从而展现为一个个icon小图标。

回头看看我们之前从[https://icomoon.io/app](https://icomoon.io/app)上下载自定义字体时，是不是也给每一个icon图标标识了一个**html实体**（e600、e601等等）？

## css样式+webfont

我们除了使用html实体配合font之外，还可以使用css样式配合font。

在[https://icomoon.io/app](https://icomoon.io/app)上下载自定义图标字体时，会同时得到相关的css代码，如下，

```css
.icon {
    font-family: 'icomoon';
    speak: none;
    font-style: normal;
    font-weight: normal;
    font-variant: normal;
    text-transform: none;
    line-height: 1;
    /* Better Font Rendering =========== */
    -webkit-font-smoothing: antialiased;
    -moz-osx-font-smoothing: grayscale;
}
.icon-home3:before {
    content: "\e600";
}
.icon-office:before {
    content: "\e601";
}
.icon-newspaper:before {
    content: "\e602";
}
.icon-pencil:before {
    content: "\e603";
}
```

所以我们只要在需要图标的html代码中添加相应的css样式即可，比如，

```html
<li>
    <a href="#"><i class="icon icon-home3"></i>主页</a>
    <a href="#"><i class="icon icon-office"></i>归档</a>
</li>
```

可见，这种方式不再需要在html代码中书写相应的html实体，因为我们给相关元素添加了伪元素，将html实体转移到了伪元素中。

这里额外提一点，为何[天猫](http://tmall.com)的html页面中使用的是html实体+font而不是css样式+font？这其实是考虑了低版本浏览器的兼容性。因为部分（低版本的）浏览器还不支持css伪元素，但是html实体一定是支持的。

## 对比

到现在为止，我们已经有了2种（严格来说，是3种）来制作和使用网页icon小图标，

- 图片 + css sprite
- webfont + html实体
- webfont + css样式

三者的对比如下图，

![](img-3-1-3.png)

总得来说，前者是操作图片，后两者操作icon小图标就跟操作字体是一样的。比如，我想缩放图标，直接修改`font-size`即可；我想修改颜色，直接修改`color`即可。除了这些简单的变化，还可以灵活的添加描边、阴影等。

不过webfont的图标色彩单一（最多也就只能做出渐变），而图片则色彩丰富。

# webfont的更多内容

## Font Awesome

[Font Awesome](http://fontawesome.io/)（[中文站点](http://fontawesome.dashgame.com/)）是一套为Bootstrap而创造的图标字体库及CSS框架，在业界享有盛誉。

FA包含了常规web开发所需要用到的几乎所有图标，并且免费授权使用，只需要下载即可。详情请参阅其官网。

## 中文WebFont

目前在国外，webfont已经非常流行，但是国内迟迟未能起步。这跟国内的环境有关系，

- 中文字体库过于庞大，需要一定带宽支持
- 国内低版本浏览器市场占有率不能忽视，但是低版本的浏览器不能良好的支持webfont

不过，目前国内已经有两家公司致力于webfont这一块的服务，

- [http://www.youziku.com/](http://www.youziku.com/ "有字库")
- [http://cn.justfont.com/](http://cn.justfont.com/ "就是字")

有翻墙资源的，可以参阅google推出的webfont云托管服务。有兴趣的同学请自行调研相关服务。

## 中文webfont压缩

[字蛛（Font-Spider）](http://font-spider.org/)是一款基于nodejs的开源的本地webfont压缩器。它的原理就是扫描页面文档，然后删除webfont中没有使用的字符，从而达到精简webfont体积的目的。有兴趣的同学可以参阅其官网。

# 参考列表

- [CSS3 @font-face](http://www.cnblogs.com/rubylouvre/archive/2011/06/19/2084875.html)
- [迟到的中文 WebFont](http://www.w3ctech.com/topic/669)
- [用字体在网页中画ICON图标](http://t.imooc.com/learn/243)

End. All rights reserved `@gejiawen`.
