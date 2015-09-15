postid: "read-cheerio-doc"
title: "通读Cheerio文档"
date: 2015-08-17 17:18:48
categories: [Nodejs]
tags: [cheerio, nodejs, 翻译, 中文文档, 漫游NodeJS系列]

---

# 前言

[cheerio](https://github.com/cheeriojs/cheerio)是一款非常实用的nodejs第三方包，适用于服务端（nodejs端）处理html。它有着与jquery及其相似（几乎是一致）的api，速度飞快，使用灵活，而且不仅能够处理html，同样也能处理xml。

本文主要的参考文档就是cheerio的[官方文档](http://cheeriojs.github.io/cheerio/)，基本上就是它的翻译。

# APIs

cheerio文档的api我将其分为以下几个方面，包括

- 加载（loading）
- 选择器（selectors）
- 属性操作（attributes）
- 结构推导（traversing）
- 结构操作（manipulation）
- 实用方法（Miscellaneous & Utilities）

在具体讲述各个api之前，我们给出一份html代码，这份html代码将会是我们下面所有api操作的示例代码。

```html
<ul id="fruits">
    <li class="apple">Apple</li>
    <li class="orange">Orange</li>
    <li class="pear">Pear</li>
</ul>
```

## 加载（loading）

在使用cheerio进行各种操作之前，我们需要首先加载一份html得到一个cherrio对象。比如

```javascript
var cheerio = require('cheerio');
var $ = cheerio.load('<ul>...</ul>');
```

因为cheerio与jquery有着极其相似的语法，所以我们一般将得到的cheerio对象命名为**`$`**，装作它就是jquery对象，反正基本上用法都一样。

除了`.load()`方法之外，我们还可以使用`$( selector, [context], [root] )`这个api来获得部分html节点作为cheerio对象。比如

```javascript
var $ = require('cheerio');
var t1 = $('ul', '<ul id = "fruits">...</ul>');
var t2 = $('li', 'ul', '<ul id = "fruits">...</ul>');
```

其中第一个参数就是我们获取的目标参数。所以`t1`得到是**`ul`**标签封装的cheerio对象，`t2`得到是3个**`li`**标签封装的cheerio对象的集合。

此外，我们在加载html时还可以设置一些配置参数，比如

```javascript
$ = cheerio.load('<ul id = "fruits">...</ul>', {
    ignoreWhitespace: true,
    xmlMode: true
});
```

关于cheerio的配置，一般我们用的较少，它默认的配置如下，

```javascript
{
    ignoreWhitespace: false, // 是否忽略空白符
    xmlMode: false, // 是否是解析xml文档
    lowerCaseTags: false // 是否采用xml模式处理。这将会影响部分tag的处理。
}
```

关于cheerio配置的更多内容，请参考[这里](https://github.com/fb55/domhandler)和[这里](https://github.com/fb55/htmlparser2/wiki/Parser-options)。


## 选择器（selectors）

cheerio的选择器基本上跟jquery拥有一致的用法。如果你熟悉jquery，那你将会倍感亲切。

```javascript
$(selector, [context], [root])
```

其中`selector`是目标选择器，`context`是目标选择器的上下文，`root`是上下文`context`的上下文。`selector`和`context`可以是**字符串表达式**、**dom元素**、**dom元素集合**、**cheerio对象**，而`root`一般都是html文档字符串。

一般地，我们通过cheerio操作html，都是以上面的这个api得到目标元素的cheerio对象开始，然后再进行各种操作。比如

```javascript
$('.apple', '#fruits').text()； //=> Apple

$('ul .pear').attr('class')； //=> pear

$('li[class=orange]').html()； //=> <li class="orange">Orange</li>
```

## 属性操作（attributes）

cheerio提供了操作元素属性的一系列方法。

### `.attr(name[, value])`

这个方法很简单，第二个参数是可选的。当只有第一个参数时表示获取属性的值，当有带有第二个参数时，表示设置属性的值。

```javascript
$('ul').attr('id'); //=> fruits

$('.apple').attr('id', 'favorite').html();
//=> <li class="apple" id="favorite">Apple</li>
```

### `.removeAttr(name)`

通过`name`移除某一个属性，同时返回被移除的这个元素。

```javascript
$('.pear').removeAttr('class').html();
//=> <li>Pear</li>
```

### `.hasClass(className)`

判断某元素的`class`中是否包含`className`。

```javascript
$('.pear').hasClass('pear'); //=> true

$('apple').hasClass('fruit'); //=> false

$('li').hasClass('pear'); //=> true
```

### `.addClass(className)`

给某元素添加一个名为`className`的样式名。

```javascript
$('.pear').addClass('fruit').html();
//=> <li class = "pear fruit">Pear</li>

$('.apple').addClass('fruit red').html();
//=> <li class = "apple fruit red">Apple</li>
```

### `.removeClass(className)`

将某元素上名为`className`的样式名移除。如果不存在`className`，则移除所有的样式名。

```javascript
$('.pear').removeClass('pear').html();
//=> <li class="">Pear</li>

$('.apple').addClass('red').removeClass().html();
//=> <li class="">Apple</li>
```

## 结构推导（traversing）

可以像使用jquery那样使用cheerio，通过某一个元素来获取它的父元素、子元素、兄弟元素等等。

### `.find(selector)`

在某元素下查询满足选择条件的元素。

```javascript
$('#fruits').find('li').length; //=> 3
```

### `.parent()`

获取某元素的父元素。

```javascript
$('.pear').parent().attr('id'); //=> fruits
```

### `.next()`

获取某元素的下一个兄弟元素。

```javascript
$('.apple').next().hasClass('orange'); //=> true
```

### `.perv()`

获取某元素的上一个兄弟元素。

```javascript
$('.orange').prev().hasClass('apple'); //=> true
```

### `.siblings()`

获取某元素的所有同级元素。（当然除了它自己）

```javascript
$('.pear').siblings().length; //=> 2
```

### `.children([selector])`

获取某元素的孩子节点。可以传入参数在所有的孩子节点中进行筛选。

```javascript
$('#fruits').children().length; //=> 3

$('#fruits').children('.pear').text(); //=> Pear
```

### `.each(function(index, element){...})`

和jquery类似的`each`迭代器，对每一个元素进行处理。

```javascript
var fruits = [];

$('li').each(function(i, elem) {
    fruits[i] = $(this).text();
});

fruits.join(', '); //=> Apple, Orange, Pear
```

### `.map(function(index, element){...})`

和jquery类似的`each`迭代器，对每一个元素进行处理并返回一个值。

```javascript
$('li').map(function(i, el) {
    // this === el
    return $(this).attr('class');
}).get().join(', '); //=> apple, orange, pear
```

### `.filter(selector)` & `.filter(function(index))`

在cheerio对象集合中进行条件筛选。

```javascript
$('li').filter('.orange').attr('class'); //=> orange

$('li').filter(function(i, el) {
  // this === el
  return $(this).attr('class') === 'orange';
}).attr('class') //=> orange
```

### `.first()`

获取cheerio集合中的第一个cheerio对象。

```javascript
$('#fruits').children().first().text(); //=> Apple
```

### `.last()`

获取cheerio集合中的最后一个cheerio对象。

```javascript
$('#fruits').children().last().text(); //=> Pear
```

### `.eq(i)`

根据索引获取cheerio集合中的某一个对象。参数可以使负数，表示从尾部开始索引。

```javascript
$('li').eq(0).text(); //=> Apple

$('li').eq(-1).text(); //=> Pear
```

## 结构操作（manipulation）

cheerio提供一系列修改dom结构的方法。

### `.append(content, [content, ...])`

将`content`插入到某元素中作为该元素的**最后一个子元素**。

```javascript
$('ul').append('<li class = "plum">Plum</li>');
$.html();
// <ul id = "fruits">
//     <li class = "apple">Apple</li>
//     <li class = "orange">Orange</li>
//     <li class = "pear">Pear</li>
//     <li class = "plum">Plum</li>
// </ul>
```

### `.prepend(content, [content, ...])`

将`content`插入到某元素中作为该元素的**第一个子元素**。

```javascript
$('ul').prepend('<li class = "plum">Plum</li>');
$.html();
// <ul id = "fruits">
//     <li class = "plum">Plum</li>
//     <li class = "apple">Apple</li>
//     <li class = "orange">Orange</li>
//     <li class = "pear">Pear</li>
// </ul>
```

### `.after(content, [content, ...])`

将`content`插入到某元素的后面，并作为其后面第一个兄弟节点。

```javascript
$('.apple').after('<li class = "plum">Plum</li>');
$.html();
// <ul id = "fruits">
//     <li class = "apple">Apple</li>
//     <li class = "plum">Plum</li>
//     <li class = "orange">Orange</li>
//     <li class = "pear">Pear</li>
// </ul>
```

### `.before(content, [content, ...])`

将`content`插入到某元素的前面，并作为其前面的第一个兄弟节点。

```javascript
$('.apple').before('<li class = "plum">Plum</li>');
$.html();
// <ul id = "fruits">
//     <li class = "plum">Plum</li>
//     <li class = "apple">Apple</li>
//     <li class = "orange">Orange</li>
//     <li class = "pear">Pear</li>
// </ul>
```

### `.remove([selector])`

移除某一个节点以及他们的孩子节点。

```javascript
$('.pear').remove();
$.html();
// <ul id = "fruits">
//     <li class = "apple">Apple</li>
//     <li class = "orange">Orange</li>
// </ul>
```

### `.replaceWith(content)`

替换匹配的节点。

```javascript
var plum = $('<li class = "plum">Plum</li>');
$('.pear').replaceWith(plum);
$.html();
// <ul id = "fruits">
//    <li class = "apple">Apple</li>
//    <li class = "orange">Orange</li>
//    <li class = "plum">Plum</li>
// </ul>
```

### `.empty()`

清空一个节点，移除其所有的孩子节点。

```javascript
$('ul').empty();
$.html();
// <ul id = "fruits"></ul>
```

### `.html([htmlString])`

获取某节点的html字符串。如果传入参数，则设置该元素的html结构。

```javascript
$('.orange').html(); //=> Orange

$('#fruits').html('<li class = "mango">Mango</li>').html();
//=> <li class="mango">Mango</li>
```

### `.text([textString])`

获取某节点的纯文本。

```javascript
$('.orange').text();
//=> Orange

$('ul').text();
//=>  Apple
//    Orange
//    Pear
```

## 实用方法（Miscellaneous & Utilities）

### `.toArray()`

将cheerio对象集合转换成真正的数据结构。

```javascript
$('li').toArray();
//=> [ {...}, {...}, {...} ]
```

### `.clone()`

克隆一个节点。

```javascript
var moreFruit = $('#fruits').clone();
```

### `$.root`

对某一cheerio对象的根节点进行相关操作。

```javascript
$.root().append('<ul id="vegetables"></ul>').html();
//=> <ul id="fruits">...</ul><ul id="vegetables"></ul>
```

### `$.contains(container, contained)`

检查`container`中是否是否包含`contained`元素。

```javascript
$.contains('#fruits', '.pear'); // => true
```




