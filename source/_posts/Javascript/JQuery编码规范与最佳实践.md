postid: 'jquery-coding-standards-and-best-practices'
title: JQuery编码规范与最佳实践
categories: [Javascript]
tags: [javascript, jquery, 翻译]
date: 2015-10-08 12:51:37

---

偶然发现我很少写jquery相关的文章。作为一名前端从业人员，jquery可以说一门必备的技能，能够熟练使用jquery是每一个前端开发者应该掌握的。不过这篇文章我并不打算写成从零开始的教程式文章，仅仅阐述一些使用jquery需要注意的一些内容。

前几日有幸读到一篇文章，讲得就是使用jquery的一些周边知识，包括一些相关的最佳实践，感觉受益匪浅。原文地址在[这里](http://lab.abhinayrathore.com/jquery-standards/)，这篇文章应该写了有一段时间了，不过可喜的是，作者并没有弃之不理，反而在最近有修改过相关内容。所以我决定索性翻译此篇文章以作备忘（本文的翻译并非完全按照原文照字照句翻译的，在某些位置做了内容合并以及应用了一些活跃的修饰手法，不过不影响原文的含义）。

译文开始。

# jquery的加载

在使用jquery时，加载jquery会有一些需要注意的小技巧，下面听我一一道来。

## 尽量使用CDN资源

使用cdn总会有一些[益处](http://www.sitepoint.com/7-reasons-to-use-a-cdn/)，而且又不要钱，不用白不用。

不过，万一你用的cdn在关键时候掉链子，那就不好玩了。所以我们还需要整点小技巧来防着这一手。

```html
<script type="text/javascript" src="//ajax.googleapis.com/ajax/libs/jquery/2.1.1/jquery.min.js"></script>

<script>
    window.jQuery || document.write('<script src="js/jquery-2.1.1.min.js" type="text/javascript"><\/script>')
</script>
```

这里稍微解释一下。上面代码中第一个`<script>`标签的作用比较显目，就是加载cdn提供的jquery资源。而第二个`<script>`标签中的代码就是为了防止cdn挂了而作出的特别处理。其中的javascript代码使用了短语判断，若浏览器环境已经存在`window.jQuery`变量则什么也不做；否则动态的插入一个`<script>`标签来加载自己服务器上的jquery资源。

所以，看起来很普通的jq文件加载过程，也是有不少门道的。

我们接着往下看。

## 使用裸协议的URL

出现了一个不太常见的词，裸协议。啥意思呢？很简单就是说去掉url中的`http:`或`https:`，就想前面的代码块中一样。关于裸协议的更多解释可以参阅[这里](http://www.paulirish.com/2010/the-protocol-relative-url/)。

> 译者注：经过译者多方资料的查阅，发现这一点不再是一个推荐的做法。如果你的页面使用的是https协议（需要SSL），那么你在html中引用外部cdn资源时，最好还是带上`https://`协议。


## 减少页面阻塞

尽可能的将你的js代码和jquery引用代码放在页面的底部。为什么应该这么做呢？简单来说，如果js代码块在html的上部的话，就会阻塞页面主体内容的加载。一般我们都是将js代码块，包括js代码的引用代码都是放在`</body>`标签之前。

关于这一点，原文给出了一篇参阅[文章](https://developer.yahoo.com/blogs/ydn/high-performance-sites-rule-6-move-scripts-bottom-7200.html)。此外，开源届还有一个非常流行的项目，[html5-boilerplate](https://github.com/h5bp/html5-boilerplate)是一个非常专业的前端起手模板，可以多参考参考。

## 如何选择版本

众所周知，jquery有很多的版本，特别地，jquery1.x.x版本和jquery2.x.x版本存在着非常大的差异。那么我们实际在生产环境中针对如此多的不同版本究竟要如何抉择呢？下面是几个推荐的考虑要素，

- 如果你必须要支持IE6、7、8，那么不好意思，最新版的jquery2.x.x跟你无缘咯。
- 如果你不需要考虑兼容性等因素，那么极力推荐你使用最新版本的jquery。
- 如果你打算使用cdn提供的jquery资源，请务必提供完整的版本，比如`1.9.2`。
- 切记莫要重复加载引用多个版本的jquery资源。
- 切记莫要在生产环境中使用cdn提供的所谓最新本jquery（[jquery-latest.js](http://blog.jquery.com/2014/07/03/dont-use-jquery-latest-js/)）

## 注意不同类库的变量冲突

如果你的项目中除了jquery之外，还使用了诸如Prototype，MooTools，Zepto等等，此时你应该需要谨慎一点，因为这些类库也使用`$`作为他们的暴露接口。此时我们可以直接使用`jQuery`。并且在存在多个类库接口冲突隐患时，我们应该调用类库提供的`$.noConflict()`来避免冲突。

## 浏览器器特性检测

因为jquery1.9.x之后的版本中，取消了`$browser`转而提供一个用于特性检测的变量`$.support`，而且jquery官方的言下之意是，这个变量是jquery内部使用的，开发者们最好不要使用，同时推荐开发者们使用[Modernizr](http://modernizr.com/)来进行特性探测。

>译者注：本博客之前也有一篇讨论浏览器特性探测的文章，若有兴趣，请移步[这里](http://gejiawen.github.io/2015/09/22/think-about-jquery-support/)。


# jquery的变量

在使用jquery变量时，我们会有一些约定成俗的不成文规定。

## 添加变量前缀

往往我们会为了效率或者访问考虑，临时缓存一些jquery变量，针对这些变量我们应该加上一个`$`前缀以示区别。

```javascript
var $myDiv = $('#myDiv');
$myDiv.click(function() {
    // TODO
});
```

## 适时缓存变量

通常我们会通过jquery选择出一个dom元素，然后对dom元素进行各式各样的操作。这里我们一般建议先将相关dom元素的jquery对象进行缓存，以便后续的各种操作，同时也有一个提升效率的因素在里面。

## 使用驼峰式命名法

在前端界，[驼峰式命名法](http://en.wikipedia.org/wiki/CamelCase)基本上得到了所有社区的认可。当你需要阅读别人代码时，驼峰式的变量命名会让你感觉更加亲切。

# 选择器

一般来说，使用jquery进行前端开发的思路都差不多。第一步选择一个dom元素，第二步对dom元素进行各式各样的操作，包括事件监听，改变dom展示等等。

jquery的核心基本上可以说就是其选择器。

## 尽量使用ID选择器

在可能的情况下，尽量使用ID选择器。因为它更快更安全。（因为其内部就是封装的原生的`document.getElementById()`，不会经过sizzle查找算法）

## 使用简单的类选择器

有的人在使用class选择器时，喜欢带上元素类型。其实这种做法**若非必要**，反而会降低选择器的效率。

```javascript
var $products = $('div.products'); // 慢
var $products = $('.products');    // 快
```

## 适时的使用`find()`

在ID容器下查找子元素时，推荐使用`find()`方法，因为这样更快。其背后的机制跟前面的**尽量使用ID选择器**基本一样。因为ID选择器不会经过sizzle的查找算法。

```javascript
var ele = $('#products div.id');        // 慢
var ele = $('#products').find('div.id') // 快
```

## 提升多级查找效率

在使用jquery进行多级dom元素查找时，应当遵循这样一个规则，即**左简右繁**。啥意思呢？让我们来看个例子。

```javascript
$('div.data .gonzalez');    // 较慢
$('.data td.gonzalez');     // 较快
```

**左简右繁**的含义，简单来说，尽量左侧的选择器简单，而越是靠右侧的选择器越要详细。

再举个具体点的例子。

```html
<div class="wrapper">
    <div class="left">
        <h2>...</h2>
        <div class="content">
            <p></p>
        </div>
    </div>
    <div class="main"></div>
    <div class="right"></div>
</div>
```

现在我若想选择`.content`下的`p`标签元素，会有许多方法。这里我们可以这么来，

```javascript
var $p = $('.wrapper .left.content.p');
```

它会比下面这种方式要更加效率，

```javascript
var $p = $('.wrapper.left.content p');
```

至于这是为什么？我这里不打算过多的阐述，不过可以指出的是，这跟css的解析优先级（css会优先解析从右侧开始解析）以及jquery的sizzle引擎有关系。读者朋友们若有兴趣可以参阅原文给出的这篇[文章](http://learn.jquery.com/performance/optimize-selectors/)或者自行查阅相关资料。

## 避免冗余选择器

有时候我们在写选择器时，会不自觉的嵌套很多层。其实一叶障目，跳出来后，你可能会发现有时候会变得很简单。

```javascript
$(".data table.attendees td.gonzalez");
$(".data td.gonzalez"); // 省略中间多余的选择器，提升效率
```

原文中还给出了简陋的[性能测试](http://jsperf.com/avoid-excessive-specificity)来说明这一点。

## 给定上下文

```javascript
$('.class'); // bad case. 因为需要变量整个dom元素来查找.class元素
$('.class', '#class-container'); // good case. 因为它不会遍历整个dom元素，仅仅在父级容器中进行查找
```

## 尽量避免使用模糊选择器

因为模糊选择器往往会带来较多的匹配消耗，从而较低了选择器效率。

```javascript
$('div.container > *'); // BAD
$('div.container').children(); // BETTER
```

## 警惕隐式的模糊选择器

大多数时候，省略的选择器其实就是`*`选择器。

```javascript
$('div.someclass :radio'); // BAD
$('div.someclass input:radio'); // GOOD
```

## 唯一的ID选择器

经常会看到一些jquery新手在使用id选择器会犯一个毛病，就是混搭id选择器和其他选择器。其实这完全没有必要。因为id选择器已经表示了唯一性。

```javascript
$('#outer #inner'); // BAD
$('div#inner'); // BAD
$('.outer-container #inner'); // BAD
$('#inner'); // GOOD, only calls document.getElementById()
```

# DOM操作

## 先卸载再操作

谨记一条，在对一个dom元素进行大量繁杂操作之前，最好将其脱离原先的dom文档，操作完毕之后再重新append上去。

至于为何要这么做。最核心的原因是防止在进行操作dom元素时导致dom树的频繁更新，甚至导致页面的重绘和重排，从而降低了页面效率。更多可参考[这里](http://learn.jquery.com/performance/detach-elements-before-work-with-them/)。

```javascript
var $myList = $("#list-container > ul").detach();
// ... 针对$myList的一系列操作
$myList.appendTo("#list-container");
```

## 减少不必要的元素操作

使用jquery时，经常会遇到需要使用手动拼接dom元素字符串的情况。这里，应该谨记一条，就是应该在所有处理操作完毕之后再append到dom树中，而不是处理一次就append一次。更多可参考[这里](http://learn.jquery.com/performance/append-outside-loop/)。

```javascript
// BAD
var $myList = $("#list");
for(var i = 0; i < 10000; i++){
    $myList.append("<li>"+i+"</li>");
}
 
// GOOD
var $myList = $("#list");
var list = "";
for(var i = 0; i < 10000; i++){
    list += "<li>"+i+"</li>";
}
$myList.html(list);
 
// EVEN FASTER
// [].join()方法会将所有的数组元素链接成一个字符串，而且速度不俗。
var array = []; 
for(var i = 0; i < 10000; i++){
    array[i] = "<li>"+i+"</li>"; 
}
$myList.html(array.join(''));
```

原文中还给出一个[性能比较](http://jsperf.com/jquery-append-vs-string-concat)。

## 避免操作不存在的元素

有时候我们通过jquery选择器拿到的jquery对象是个空数组(`[]`)，可能是那一块内容还未生成或者其他什么原因。此时我们不应该对一个空的jquery对象进行操作。详情可参考[这里](http://learn.jquery.com/performance/dont-act-on-absent-elements/)。

```javascript
// BAD: This runs three functions before it realizes there's nothing in the selection
// jquery需要运行3个函数，才会知道选择器什么也没拿到。
$("#nosuchthing").slideUp();

// GOOD
var $mySelection = $("#nosuchthing");
if ($mySelection.length) {
    $mySelection.slideUp();
}
```

# 事件

## 一个文档ready处理程序

一个页面只写一个文档ready事件的处理程序。这样代码既清晰好调试，又容易跟踪代码的进程。

## 慎用匿名函数作为回调

匿名函数往往会造成一些不便之处，比如不利于调试、维护、测试、复用等等。更多内容可参考[这里](http://learn.jquery.com/code-organization/beware-anonymous-functions/)。

```javascript
$("#myLink").on("click", function(){...}); // BAD
 
// GOOD
function myLinkClickHandler(){...}
$("#myLink").on("click", myLinkClickHandler);
```

同时，处理文档ready的事件回调函数也尽量别用匿名函数。

```javascript
$(function(){ ... }); // BAD: You can never reuse or write a test for this function.
 
// GOOD
$(initPage); // or $(document).ready(initPage);
function initPage(){
    // Page load event where you can initialize values and call other initializers.
}
```

除此之外，处理文档ready的事件回调函数应该使用外部文件引入的方式，而且页面中嵌入初始化调用即可。

```javascript
<script src="my-document-ready.js"></script>
<script>
	// Any global variable set-up that might be needed.
	$(document).ready(initPage); // or $(initPage);
</script>
```

> 译者注：关于这一块内容，译者是不太赞同原作者的观点的。

## 尽量避免在html中内联js代码

html允许直接在html标签的属性中添加js相关的逻辑处理代码。这虽然带来了方便，但是更多的是给调试带来了不便。我们应该总是走jquery先选择后绑定这个路子。

```html
<a id="myLink" href="#" onclick="myEventHandler();">my link</a> <!-- BAD -->
```

```javascript
$("#myLink").on("click", myEventHandler); // GOOD
```

## 事件命名空间

如果可以，我们可以设计一套专门用于区分事件的[命名空间](http://api.jquery.com/event.namespace/)。这样利于维护且不会对其他事件造成影响。

```javascript
$("#myLink").on("click.mySpecialClick", myEventHandler); // GOOD
// Later on, it's easier to unbind just your click event
$("#myLink").unbind("click.mySpecialClick");
```

## 使用事件代理

当你需要给相似的元素添加同一个事件处理时，你就需要用到jquery的[事件代理](http://learn.jquery.com/events/event-delegation/)了。所谓事件代理，说简单点就是给系列元素的父元素添加事件监听，而且事件真正触发在其子元素上。

```javascript
$("#list a").on("click", myClickHandler); // 不好。因为相当于添加了多次相同的事件
$("#list").on("click", "a", myClickHandler); // 不错。使用事件代理，在父元素上添加事件，在子元素上触发
```

# Ajax异步操作

jquery内置了ajax方法，提供XMLHTTPRequest服务，并且屏蔽了不同浏览器对XMLHTTPRequest的差异支持。

## 尽量使用`$.ajax()`

有的人喜欢使用`$.get()`，`$.getJson()`等方法，不过jquery内部任然是将其转换成`$.ajax()`

> 译者注：关于这一点，译者是不太赞同原作者的观点的。

## 请求可以省略协议

在https站上不要使用http请求，最好是请求时别指定请求协议。（将http或者https从你的url中移除）

## 正确传递参数

不要在请求url中附带参数。`$.ajax()`提供有专门参数用于传递参数。后者更加可读。

```javascript
// Less readable...
$.ajax({
    url: "something.php?param1=test1&param2=test2",
    ....
});
 
// More readable...
$.ajax({
    url: "something.php",
    data: { param1: test1, param2: test2 }
});
```

## Ajax杂项

发起ajax请求时，最好都指定请求返回的数据类型（dataType）。

使用事件代理，以便ajax动态加载出来的内容仍然能够触发之前的事件绑定。更多内容可参考[这里](http://lab.abhinayrathore.com/jquery-standards/)。

```javascript
$("#parent-container").on("click", "a", delegatedClickHandlerForAjax);
```

使用Promise模式来处理ajax不同情况的返回。点我查看更多的[示例](http://www.htmlgoodies.com/beyond/javascript/making-promises-with-jquery-deferred.html)。

```javascript
$.ajax({ ... }).then(successHandler, failureHandler);
 
// OR
var jqxhr = $.ajax({ ... });
jqxhr.done(successHandler);
jqxhr.fail(failureHandler);
```

## 通用ajax模板

下面是一个通用的使用jquery发送ajax请求的模板。更多的说明可参考[这里](https://api.jquery.com/jQuery.ajax/)。

```javascript
var jqxhr = $.ajax({
    url: url,
    type: "GET", // 默认是GET，不过使用其他http动词
    cache: true, // 默认是true，不过当dataType为script和jsonp时为false
    data: {}, // 请求参数
    dataType: "json", // 最好指明请求返回的数据类型，默认为json
    jsonp: "callback", // 指定回调处理JSONP类型的请求
    statusCode: { // 如果你想处理其他的错误，可以在这里指明各错误码对应的回调函数
        404: handler404,
        500: handler500
    }
});
jqxhr.done(successHandler);
jqxhr.fail(failureHandler);
```

# 动画与特效

下面是两点关于jquery动画的通用建议。

1. 尽量让所有的动画和特效风格如一，不要让人感觉头晕目眩
2. 切记滥用动画和特效，用户体验至上
    1. 使用简洁的显示隐藏，状态切换，滑入滑出等效果来展示元素
    2. 使用预设值来设置动画的速度'fast'，'slow'，或者400（中等速度）

# 插件

1. 始终选择一个有良好支持，完善文档，全面测试过并且社区活跃的插件
2. 注意所用插件与当前使用的jQuery版本是否兼容
3. 一些常用功能应该写成jQuery插件

# 链式语法

jquery支持链式语法。很多时候我们可以使用链式语法将一系列操作串起来。

除了使用临时变量缓存jquery对象之外，我们还可以使用链式调用。

```javascript
$("#myDiv").addClass("error").show();
```

此外，当链式调用多达3次以上或代码因绑定回调略显复杂时，使用换行和适当的缩进来提高代码的可读性。

```javascript
$("#myLink")
    .addClass("bold")
    .on("click", myClickHandler)
    .on("mouseover", myMouseOverHandler)
    .show();
```

当然，针对特别长的链接调用，最好还是使用临时变量缓存一下的好。

# 其他

## 使用对象直接量传递参数

```javascript
// BAD, 调用了3此attr()方法
$myLink.attr("href", "#").attr("title", "my link").attr("rel", "external"); 

// GOOD, 只调用了一次attr()方法
$myLink.attr({
    href: "#",
    title: "my link",
    rel: "external"
});
```

## 不要将css揉进jquery中

推荐的方式是，以css class为单位去操作dom元素，而不是直接在dom元素上添加移除css属性。

```javascript
$("#mydiv").css({'color':red, 'font-weight':'bold'}); // BAD
```

```css
error { color: red; font-weight: bold; } /* GOOD */
```

```javascript
$("#mydiv").addClass("error"); // GOOD
```

## 最好别用已被官方废弃的方法

这点没什么好说的。我们需要关注所使用的具体版本以及官方的changlog即可。这里有一份废弃方法的[列表](http://api.jquery.com/category/deprecated/)。

## 适时的使用原生js

在适当或者需要的时候，还是可以使用一些原生js代码的。这里有一份原文给出的[相关性能测试](http://jsperf.com/document-getelementbyid-vs-jquery/3)。

> 译者注：关于这一点，大家看看就好了，不必较真。在一些js代码量较大的项目中，是不太会允许这里一坨原生代码那里一坨原生代码的。不然维护的成本会越来越大。性能这一块，其实说到底还是得看场景和需求，再一个就是平衡点的把握。


# 参考列表

- jQuery Performance: [http://learn.jquery.com/performance/](http://learn.jquery.com/performance/) 
- jQuery Learn: [http://learn.jquery.com](http://learn.jquery.com)
- jQuery API Docs: [http://api.jquery.com/](http://api.jquery.com/) 
- jQuery Coding Standards and Best Practice: [http://www.jameswiseman.com/blog/2010/04/20/jquery-standards-and-best-practice/](http://www.jameswiseman.com/blog/2010/04/20/jquery-standards-and-best-practice/) 
- jQuery Cheatsheet: [http://lab.abhinayrathore.com/jquery-cheatsheet/](http://lab.abhinayrathore.com/jquery-cheatsheet/)
- jQuery Plugin Boilerplate: [http://stefangabos.ro/jquery/jquery-plugin-boilerplate-revisited/](http://stefangabos.ro/jquery/jquery-plugin-boilerplate-revisited/) 


