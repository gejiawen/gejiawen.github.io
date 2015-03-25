title: 深入理解JavaScript系列（7）-根本就没有“JSON对象”这回事！
date: 2014-12-30 19:25:03
categories: [Javascript]
tags: [深入理解JavaScript系列]
---

本文是[深入理解JavaScript系列](http://gejiawen.github.io/2014/11/13/%E7%B3%BB%E5%88%97/%E6%B7%B1%E5%85%A5%E7%90%86%E8%A7%A3JavaScript%E7%B3%BB%E5%88%97/)的第**七**篇读文笔记，博客原文在[这里](http://www.cnblogs.com/TomXu/archive/2012/01/11/2311956.html)。

# 内容简要

鉴于JSON数据格式的越来越流行，但是**JSON对象**和**JSON字符串**这两种概念往往又会被人混淆。所以本篇文章就是对相关内容的阐述。

# BACKBONE

## 数据类型的分类

现在比较流行的网络数据交换格式包括：[JSON](http://www.json.org/json-zh.html)，[YAML](http://www.yaml.org/)以及日薄西山的[XML](http://en.wikipedia.org/wiki/XML)。

从结构上看，所有的数据最后都可以分解成三个基本类型，

1. 标量（scalar），也就是一个单独的字符串（String）或者数字（Number）。比如"beijing"这个单独的词。
2. 序列（sequence），也就是若干个相关的数据按照一定的顺序组合在一起。又叫做数组（Array）或者列表（List）。比如"beijing,shanghai"。
3. 映射（mapping），也就是一个键值对（key:value），即数据有一个名称，还有一个与之对应的值。这个又称作散列（Hash）或者字段（Dictionary）。比如"首都:北京"。

此部分内容参考自[数据类型和Json格式](http://www.ruanyifeng.com/blog/2009/05/data_types_and_json.html)。

可见数据构成的最小单位是非常简单的，而JSON正式完美的符合了这三种最基本的数据构成单元。

## JavaScript中字面量的含义

在JavaScript中，我们经常会听到这样一个词，JSON字面量，对象字面量，字符串字面量等等，他们究竟是什么意思呢？

引用[developer.mozilla.org](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide)中的说法，字面量就是

1. 固定的值，让你从“字面上”理解其含义。
2. 字符串字面量是由双引号（`""`）或者单引号（`''`）括起来的零个或多个字符。
3. 对象字面量是由大括号（`{}`）括起来，拥有零个或多个键值对。

比如下面的代码，

```javascript
// 这些都是字面量
var a = 1;
var name = 'puck';
var puck = {
    name: 'puck',
    age: 20
};
```

## JSON字符串和JSON对象

首先，JSON有非常严格的语法，如下，

```json
{
    "key": "value"
}
```

其中，键和值都**必须要用双引号**括起来，不用或者使用单引号都是不行滴。

从某种意义上说，JSON字符串终归是字符串，也就是说JSON字符串是字符串的子集，不过JSON字符串满足这样的条件： **JSON字符串可以被正确的反序列化成JSON对象**。

而JSON对象与普通的JavaScript对象却不是那么好区分，要看对象所在的上下文场景。看下面的代码，

```javascript
// JSON字符串
var puck = '{"name": "puck", "age": 20}';

// 字面量对象，这里我们可以简单将其看成是一个JSON对象
var puck2 = {
    "name": "puck2",
    "age": 20
};
```

## 真正的JSON对象

从严格意义上来说，满足JSON语法的普通JavaScript字面量对象并不是JSON对象。两者是完全不一样的概念。

在新版的浏览器里JSON对象已经被原生的内置对象了，目前有2个静态方法：`JSON.parse`用来将JSON字符串反序列化成对象，`JSON.stringify`用来将对象序列化成JSON字符串。老版本的浏览器不支持这个对象，但你可以通过[json2.js](http://json.org/)来实现同样的功能。

# 总结

好吧，这篇其实算是一篇水文，讲述的内容比较简单，算是一篇科普文吧。

这里只想强调一条，**JSON是一种对数据结构的简洁描述**，然后还要知道现代浏览器中已经有`JSON.parse`和`JSON.stringify`这两个方法就可以了。

End. All rights reserved `@gejiawen`.
