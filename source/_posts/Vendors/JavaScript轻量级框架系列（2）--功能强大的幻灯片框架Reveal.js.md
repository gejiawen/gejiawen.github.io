title: JavaScript轻量级框架系列（2）--功能强大的幻灯片框架Reveal.js
date: 2014-11-30 15:13:55
categories: [Vendors]
tags: [JavaScript轻量级框架系列, html, javascript]
---

本文是[JavaScript轻量级框架系列](http://gejiawen.github.io/2014/11/26/%E7%B3%BB%E5%88%97/JavaScript%E8%BD%BB%E9%87%8F%E7%BA%A7%E6%A1%86%E6%9E%B6%E7%B3%BB%E5%88%97/)的第**二**篇文章。

**reveal.js**是一款开源的、功能强大的JavaScript幻灯片框架，通过简单的HTML就能够创建非常漂亮的幻灯片。其在github上有18000+个star，可见这个JavaScript框架的活跃程度和质量。

- [github地址](https://github.com/hakimel/reveal.js)
- [live demo](http://lab.hakim.se/reveal-js/)或者[template demo](http://gejiawen.github.io/slides/a-slides-template)（前者部分资源可能需要翻墙）


# 基本结构

Reveal.js的基本结构其实很简单，它实质上就是一个html页面，不过这个html的核心内容部分需要按照reveal.js规定好的套路来，不然它就解析不出来了。其实大部分的JavaScript幻灯片框架都是一个原理，只不过各个框架的实现细节和复杂程度不一样，包括本系列[上一篇文章](http://gejiawen.github.io/2014/09/23/vendors/JavaScript%E8%BD%BB%E9%87%8F%E7%BA%A7%E6%A1%86%E6%9E%B6%E7%B3%BB%E5%88%97%EF%BC%881%EF%BC%89--%E8%B6%85%E7%82%AB3D%E6%95%88%E6%9E%9C%E7%9A%84%E5%B9%BB%E7%81%AF%E7%89%87%E6%A1%86%E6%9E%B6Impress.js/)阐述的Impress.js。

下面让我们一起来看看Reveal.js指定的游戏规则具体是什么样子的，

```html
<html>
    <head>
        <!-- 添加meta及css引入 -->
    </head>
    <body>
        <div class="reveal">
            <div class="slides">
                <section>
                    <!-- 第一张幻灯片内容 -->
                </section>
                <section>
                    <!-- 第二张幻灯片内容 -->
                </section>
                <!-- 更多的幻灯片 -->
            </div>
        </div>

        <!-- 引入reveal.js文件 -->
        <script>
            // reveal.js的初始化配置，包括一些插件的加载
        </script>
    </body>
</html>
```

从上面的代码片段可以看出，html的文件结构其实很简单。核心就是下面html片段包裹的区域，

```html
<div class="reveal">
    <div class="slides">...</div>
</div>
```

其中，每一个`<section>`都是一张幻灯片。[live demo](http://lab.hakim.se/reveal-js/)中效果绚丽的幻灯片其实就是对这些`<section>`标签耍了些手段，让他们具备了绚丽效果的展示。

需要特别说明的一点就是，上面的所有`<section>`标签都是依次出现的，并没有出现嵌套的情况（即`<section>`标签中嵌套了`<section>`标签），它的展现效果是，所有的幻灯片都依次从左到右的展示。


# 功能要点

Reveal.js真的是一款功能强大的幻灯片框架，基本上涵盖了其他幻灯片框架的所有功能。而且采用插件式设计，将不同的功能封装到不同的插件，按需加载，保证了其灵活性。同时拥有生动的幻灯片过渡效果，绚丽的内置主题，以及一些完善的功能设计，能够让你的幻灯片更加完美的展示。

下面我们来谈谈其主要的几个特性。

## 幻灯片嵌套

html代码中如果出现`<section>`标签嵌套的情况，reveal.js将会如何表现呢？假设我们有如下的html结构，

```html
<div class="reveal">
    <div class="slides">
        <section>Single Horizontal Slide</section>
        <section>
            <section>Vertical Slide 1</section>
            <section>Vertical Slide 2</section>
        </section>
    </div>
</div>
```

可见第二个`<section>`标签中又嵌套了两个`<section>`标签，这样一来第二张幻灯片其实就是由两张子幻灯片组成的，在方向上是垂直的。具体效果可以体验[Live Demo](http://lab.hakim.se/reveal-js/)或者[template demo](http://gejiawen.github.io/labs/res/a-slides-template)。

这其实是一个很有意思的特性，当我们一张幻灯片中不能满足一个主题的描述时，可以采用子幻灯片的形式来补充，此时幻灯片的嵌套将会非常有用。

## Markdown语法支持

我本人比较喜欢markdown这种简洁的标记式语法。在接触到reveal.js之前，我曾特意去搜索了一些能够直接解析markdown文件并生成幻灯片的JavaScript框架，的确得到了一些结果，比如[cleaver](https://github.com/jdan/cleaver)，就使用上来说，的确非常简单，能够非常快速的生成幻灯片页面。但是这个轻量的框架不能满足这样一个需求：不太容易生成图文并茂的幻灯片，一般只能生成一些简单的纯文字幻灯片，而且展示效果不是那么的优美。

reveal.js是支持markdown语法的，而且有两种不同形式的支持。

### 内联片段支持

我们可以在某一个`<section>`标签中（即某一张幻灯片中）插入一段markdown代码片段，如下所示，

```html
<section data-markdown>
    <h2>内联片段支持</h2>

    <script type="text/template">
        ## Page title

        A paragraph with some text and a [link](http://hakim.se).
    </script>
</section>
```

这里，我们有两点需要注意，

- `<section>`标签中需要添加额外的属性标签`data-markdown`来标识此幻灯片中有markdown代码片段
- markdown的代码片段需要被`<script type="text/template">`标签包裹，可见markdown在其中被当作成一种模版

### 外部文件支持

除了支持内联的markdown代码片段之外，reveal.js还可以解析一个外部引入的markdown文件，如下所示，

```html
<section data-markdown="example.md"
         data-separator="^---"
         data-vertical="^--"
         data-notes="^Note:"
         data-charset="utf-8">
</section>
```

从上面的html代码中，我们添加几个`data-*`属性，他们的作用分别如下，

- `data-markdown="example.md"`  需要解析的外部markdown文件路径
- `data-separator="^---"`   表明markdown文件中使用`---`符号来区分不同的幻灯片
- `data-vertical="^--"` 表明markdown文件中使用`--`符号来区分垂直子幻灯片
- `data-notes="^Note:"` 演讲者备注的标识
- `data-charset="utf-8"`    markdown文件的编码格式


对应的markdown文件内容范例如下，

```markdown
## 我是标题一

我是内容一

--

### 我是副标题一

我是内容

--

### 我是副标题二

我下面是代码

var p = {
    name: 'gejiawen',
    age: 18
};
console.log('hello world', p.name);

--

### 我是副标题三

- Item 1 <!-- .element: class="fragment" data-fragment-index="1" -->
- Item 2 <!-- .element: class="fragment" data-fragment-index="2" -->
- Item 3 <!-- .element: class="fragment" data-fragment-index="3" -->
- Item 4 <!-- .element: class="fragment" data-fragment-index="4" -->
- grow <!-- .element: class="fragment grow" data-fragment-index="5" -->
- roll-in <!-- .element: class="fragment roll-in" data-fragment-index="6" -->
- highlight-current-blue <!-- .element: class="fragment highlight-current-blue" data-fragment-index="7" -->

--

### 我是副标题四

<!-- .slide: data-background="#007777" -->
我是内容

---

## 我是第二页幻灯片

<!-- .slide: data-transition="linear" data-background="#4d7e65" data-background-transition="slide" -->
我是内容

---

## 我是第三页幻灯片

<!-- .slide: data-transition="linear" data-background="#8c4738" data-background-transition="zoom" -->
我是内容
```

从上面的markdown内容，我们可以看出，

- 总共有三页幻灯片
- 第一页幻灯片有4个子幻灯片

看到了吗，就是这么简单。

### 对markdown的额外支持

在上面小节中的markdown代码中，如果你观察仔细的话，可能会看到一些奇怪的代码，比如，

```markdown
### 我是副标题三

- Item 1 <!-- .element: class="fragment" data-fragment-index="1" -->
- Item 2 <!-- .element: class="fragment" data-fragment-index="2" -->
- Item 3 <!-- .element: class="fragment" data-fragment-index="3" -->
- Item 4 <!-- .element: class="fragment" data-fragment-index="4" -->
- grow <!-- .element: class="fragment grow" data-fragment-index="5" -->
- roll-in <!-- .element: class="fragment roll-in" data-fragment-index="6" -->
- highlight-current-blue <!-- .element: class="fragment highlight-current-blue" data-fragment-index="7" -->

--

### 我是副标题四

<!-- .slide: data-background="#007777" -->
我是内容
```

发现了吗？

这些`<!-- .element: class="fragment" data-fragment-index="1" -->`代码，类似html的注释语法是干啥的啊？

其实这些额外的属性是为了增强markdown的表现性。主要分为两类，

- `<!-- .element: xxx -->`  给某个元素添加额外的表现特性，比如fragment等
- `<!-- .slide: xxx -->`  给某个slide（幻灯片）添加额外的表现特性，比如背景色，内容变换效果等。

我个人觉得这是个非常有用的设计，很大程度上弥补了markdown生成的幻灯片表现性不强的缺陷。

还有一点需要特别指出的是，reveal.js对外部markdown文件的解析，**需要web server的支持**，因为是通过服务器去拿到相应的markdown文件的。

## 初始化及配置

### 配置

在之前的**基础结构**中可以看到，reveal.js的初始化及相关配置是位于整个html文件的最后部分的。下面给出其常用的一些配置项，

```javascript
// 下面的值都是reveal.js的默认值
Reveal.initialize({
    // 是否展示导航控制器
    controls: true,
    // 是否展示幻灯片进度条
    progress: true,
    // 是否展示当前幻灯片的页面
    slideNumber: false,
    // 是否启用浏览器的历史功能
    history: false,
    // 是否启用键盘快捷键
    keyboard: true,
    // 是否启用幻灯片鸟瞰模式
    overview: true,
    // 是否让幻灯片垂直居中
    center: true,
    // 是否启用移动的触控响应
    touch: true,
    // 是否开启幻灯片循环（最后一张结束后继续下一张跳转回第一张）
    loop: false,
    // 是否启用fragments功能
    fragments: true,
    // 是否自动轮播。0为不自动轮播，不为0的值即为自动轮播的间隔。
    autoSlide: 0,
    // 当用户输入则禁用自动轮播
    autoSlideStoppable: true,
    // 是否使用鼠标滚轮进行幻灯片播放
    mouseWheel: false,
    // 幻灯片变换动画类型，可选值 default/cube/page/concave/zoom/linear/fade/none
    transition: 'default',
    // 变换动画速度，可选值 default/fast/slow
    transitionSpeed: 'default',
    // 幻灯片背景变换风格，可选值 default/none/slide/concave/convex/zoom
    backgroundTransition: 'default',
});
```

一般来说，大部分的配置属性都是直接使用默认值即可，想了解更多信息，请阅读其[文档](https://github.com/hakimel/reveal.js/blob/master/README.md)

### 依赖

前面说过reveal.js将各种不同的功能都封装成了独立的插件，在实际使用的过程中按需加载即可。如下，

```javascript
Reveal.initialize({
    dependencies: [
        // 一个跨浏览器的获取classList的解决方案插件
        {
            src: 'lib/js/classList.js',
            condition: function() {
                return !document.body.classList;
            }
        },

        // markdown解析支持插件
        {
            src: 'plugin/markdown/marked.js',
            condition: function() {
                return !!document.querySelector('[data-markdown]');
            }
        },
        {
            src: 'plugin/markdown/markdown.js',
            condition: function() {
                return !!document.querySelector('[data-markdown]');
            }
        },

        // 代码高亮插件
        {
            src: 'plugin/highlight/highlight.js',
            async: true,
            callback: function() {
                hljs.initHighlightingOnLoad();
            }
        },

        // 幻灯片局部区域的缩放插件
        {
            src: 'plugin/zoom-js/zoom.js',
            async: true,
            condition: function() {
                return !!document.body.classList;
            }
        },

        // 演讲者备注插件
        {
            src: 'plugin/notes/notes.js',
            async: true,
            condition: function() {
                return !!document.body.classList;
            }
        },

        // 远程控制插件
        {
            src: 'plugin/remotes/remotes.js',
            async: true,
            condition: function() {
                return !!document.body.classList;
            }
        },

        // 常用数学公式表达式插件
        {
            src: 'plugin/math/math.js',
            async: true
        }
    ]
});
```

一般来说，`classList`和`highlight`是必用的插件，如果你用到了markdown，就需要加载markdown插件。当然这些都是根据不同需求而定的。想了解更多信息，请阅读其[文档](https://github.com/hakimel/reveal.js/blob/master/README.md)。

## 幻灯片添加额外特性

还记得吗，我们前面提到了**对markdown的额外支持**，其实它更一般的使用场景是直接在`<section>`标签上添加各种属性，如下，

```html
<section data-background="#ff0000">
    <h2>All CSS color formats are supported, like rgba() or hsl().</h2>
</section>
<section data-background="http://example.com/image.png">
    <h2>This slide will have a full-size background image.</h2>
</section>
<section data-background="http://example.com/image.png" data-background-size="100px" data-background-repeat="repeat">
    <h2>This background image will be sized to 100px and repeated.</h2>
</section>
```

这里唯一值得一提就是，在`<section>`标签上添加的额外属性会覆盖`Reveal.initialize()`时的配置。

## fragments

Fragments是用来突出显示幻灯片上的某一个元素的。[体验一下](http://gejiawen.github.io/labs/res/a-slides-template/#/fragments)。

其实实现起来很简单，只需要给你想要fragment的元素添加一个*css class name*即可。如下，

```html
<section>
    <p class="fragment">normal</p>
    <p class="fragment grow">grow</p>
    <p class="fragment shrink">shrink</p>
    <p class="fragment roll-in">roll-in</p>
    <p class="fragment fade-out">fade-out</p>
    <p class="fragment current-visible">visible only once</p>
    <p class="fragment highlight-current-blue">blue only once</p>
    <p class="fragment highlight-red">highlight-red</p>
    <p class="fragment highlight-green">highlight-green</p>
    <p class="fragment highlight-blue">highlight-blue</p>
</section>
```

更加高级一点的用法，

```html
<section>
    <span class="fragment fade-in">
        <span class="fragment fade-out">I'll fade in, then out</span>
    </span>
</section>
```

这里有两个地方需要注意，

- 如果只加`fragment`类，将会使元素变为不可见。
- `fragment`可以很多效果能够联合使用，如`grow`等。此时一般来说元素的可见的。
    - 有一种情况是例外，即`fragment`和`fade-in`等某些具有让元素隐藏的*css class name*联合使用时。如上面的高级用法。
- 还可以使用`data-fragment-index`属性来指定各元素的执行次序

## 内部跳转

我们知道reveal.js将每一个`<section>`标签解析成一个幻灯片。reveal.js内部会**依次**给每个`<section>`标签添加一个数字id，在浏览幻灯片时，浏览器的url如下，

```shell
http://localhost:4000/livedemo/#/3
```

如果当前的幻灯片正处在一个子幻灯片上，那么浏览器的url如下，

```shell
http://localhost:4000/livedemo/#/3/1
```

如果你给`<section>`标签添加了`id`属性，比如

```html
<section id="test"></section>
```

那么，浏览器的url就会如下，

```shell
http://localhost:4000/livedemo/#/test
```

根据上面的分析，我们可以在某一张幻灯片中添加一个内部跳转链接，比如

```html
<a href="#/2/2">Link</a>
<a href="#/some-slide">Link</a>
```

这样我们就可以直接进行内部跳转了。

## 其他功能

除了上面介绍的各种功能和特点之外，reveal.js仍然还有不少可以挖掘的地方，比如**事件机制**、**PDF导出支持**、**演讲备注**、**同步演示**等。这里就不多作描述了，想了解更多信息，请阅读其[文档](https://github.com/hakimel/reveal.js/blob/master/README.md)。

# 总结

Reveal.js的确是一款功能强大的幻灯片框架，深受web开发者的喜爱。基本上可以非常轻松的制作出一个幻灯片。特别针对那些Geekers来说，如果只是一些纯文字的描述，那么一份markdown文件将会帮你搞定所有的事情。即使是需要制作图文并茂的幻灯片，也不是难事，无非就是多花费些时间，对`<section>`标签添加一些额外的属性而已。

你还在执着office的ppt么？来试试reveal.js吧。

End. All rights reserved `@gejiawen`.
