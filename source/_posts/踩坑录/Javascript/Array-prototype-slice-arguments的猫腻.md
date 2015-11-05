postid: 'perf-about-array-prototype-slice-arguments'
title: Array.prototype.slice(arguments)的猫腻
categories: [踩坑录]
tags: [javascript]
date: 2015-11-04 11:25:43

---

# 前言

在我博客之前有过这样一篇文章，[JS中易混淆方法备忘录](http://gejiawen.github.io/2015/04/02/confused-methods-in-javascript)。其中有介绍可以利用一种约定技巧将类数组对象转换成对象。如下

```javascript
function foo() {
    console.log(arguments);
    var args = Array.prototype.slice.call(arguments);
    console.log(args);
    // more logic
}
```

在javascript中，函数中的`arguments`是一个特殊的内置变量，它其实是一个对象，但是其内部的表现却跟数组一致。如下图，

![](/res/perf-about-array-prototype-slice-arguments/001.png)

其中`arguments.callee`和`arguments.Symbol`属性都是不可枚举的（无法用`for...in`遍历）。

在经过`var args = Array.prototype.slice.call(arguments)`操作之后，从上图可见，`args`的结果是一个数组。

# 原理

如果你不清楚上面内容的内幕，你可能禁不住要问个为什么！下面我们来研究一下其中的内幕。

首先，在Javascript中有两个`slice`方法，分别是`Array.prototype.slice`和`String.prototype.slice`。如果你对`xxx.prototype.xxx`这种写法有疑问，我想你可能需要补充一下Javascript中原型相关的知识了（无节操小广告：[推荐文章1](http://gejiawen.github.io/2014/09/29/ecmascript-inherit/)，[推荐文章2](http://gejiawen.github.io/2014/10/16/prototype-inherit-in-javascript/)，[推荐文章3](http://gejiawen.github.io/2015/03/18/different-from-proto-and-prototype/)）。这里我们要说的就是前者。

我们先来看几个例子，

```javascript
var t1 = { 0: 'a', 1: 'b', 2: 'c', length: 3 };
var t2 = { 0: 'a', 1: 'b', 2: 'c', length: 3, 'name': 'larry ge' };
var t3 = { 0: 'a', 1: 'b', length: 3 };
var t4 = { length: 2 };
var t5 = { 0: 'a', 1: 'b' };

var slice = [].slice;

console.log(slice.call(t1)); // ['a', 'b', 'c']
console.log(slice.call(t2)); // ['a', 'b', 'c']
console.log(slice.call(t3)); // ['a', 'b']
console.log(slice.call(t4)); // []
console.log(slice.call(t5)); // []
```

可见，我们只能转化对象中**以数字为键值**的属性，同时必须要求对象中有值为数字的`length`属性。

到这里，我们可以猜想一下`slice`内部大概是如何实现的了。

```javascript
Array.prototype.slice = function(start, end) {
    var result = new Array();
    start = start || 0;
    end = end || this.length; // 这里的this指代当前调用此slice方法的数组，如果使用call或者apply，这个this将会被重定向。这也就是为何在没有end参数的情况下，必须要有length属性的原因
    for(var i = start; i < end; i++){
        // result.push(this[i]);
        result[i] = this[i]; // 这里使用的是this[i]，所以前面示例中的对象中那些不是以数字为键的属性都是拿不到的
    }
    return result;
}
```

所以当我们使用`Array.prototype.slice.call`或者`Array.prototype.slice.apply`时，将会把函数中的`this`自动替换成我们传入的对象。函数执行完毕之后，我们就会得到一个数组了。

以上就是这种技巧的原理了。

衍生的，我们一般会有如下三种方法将一个类数组对象转换成真正的数组，

```javascript
// 方法一
var args = Array.prototype.slice.call(arguments);

// 方法二
var args = [].slice.call(arguments, 0);

// 方法三
var args = []; 
for (var i = 0; i < arguments.length; i++) { 
    // args.push(arguments[i]);
    args[i] = arguments[i]
}
```

# 猫腻

当使用`[].slice.call`将类数组转换数组成为一种普遍技巧时（我也是一直这么用的），我觉得这是理所当然的，应该不会有什么问题。直到我前段时间读[tj/node-thunkify](https://github.com/tj/node-thunkify)源码时，突然发现一个很有意思的问题。

node-thunkify是一个用来将一个普通函数转化成thunk函数的，它的[源码](https://github.com/tj/node-thunkify/blob/master/index.js)其实很简单，其中有一段，

```javascript
// more code
var args = new Array(arguments.length);

for(var i = 0; i < args.length; ++i) {
    args[i] = arguments[i];
}
// more code
```

当时我就奇怪了，这里怎么使用for循环遍历arguments对象，而不是使用一贯的转换技巧呢？难道有什么猫腻？

果然不出所料，我在node-thunkify的issue列表中也找到了跟我有着相关疑问的同学提出的一个[issue](https://github.com/tj/node-thunkify/issues/13)。然后经过我一番资料的查阅才知道，原来这里还有这么多的门门道道。

首先通过github的issue，我在stackoverlfow上找到了一些相关的问题，比如[这个](http://stackoverflow.com/questions/23509312/does-node-js-really-not-optimize-calls-to-slice-callarguments)。题主这个问题的本意是想问“难道Node.js真的没有对`[].slice.call(arguments)`作任何的优化操作么？”

这个问题中引出了几个其他的问题，

- Node.js的V8引擎会对一些被称之为“hot function”的代码块进行优化
- 有一些代码的写法会导致V8无法进行优化
- 当操作`arguments`对象时，需要额外的注意，因为稍不注意就会导致V8无法优化

Node.js代码库的核心开发者[isaacs](https://github.com/isaacs)为此写了一篇[文章](http://blog.izs.me/post/7746314700/benchmark-array-ification-of-arguments)，详细说明了在Node.js中应该如何将类数组对象`arguments`处理成一个真正的数组。

这篇文章总结下来，就是两点需要注意，

- 如果函数的参数不确定（数量及类型），那么不推荐你定义任何的形式参数。
- 如果想从`arguments`中取出参数。那么推荐你按照如下的方式来做。

```javascript
function varArgsList () {
    var args = arguments.length === 1 ? [arguments[0]] : Array.apply(null, arguments);
    // now args is a array
}
```

或者你可能想要获取除了第一个参数之后的所有参数（这种情况常在命令行工具中遇到），

```javascript
function manualMap () {
    var l = arguments.length;
    var arr = new Array(l - 1);
    
    for (var i = 1; i < l; i ++) {
        arr[i - 1] = arguments[i];
    }
}
```

同时，Node.js的代码库中也有关于这方面的[改动](https://github.com/nodejs/node-v0.x-archive/commit/91f1b250ecb4fb8151cd17423dd4460652d0ce97)，主要是将`EventEmitter.prototype.emit`中抽取`arguments`的实现方式改变了。因为`EventEmitter`模块可以说是Node.js中最重要的模块之一，是所有事件驱动解决方案的基石，无疑`EventEmitter`就是所谓的“hot function”。


[这里](http://jsperf.com/213213213)是一些相关的benchmark。

[BlueBird的文档](https://github.com/petkaantonov/bluebird/wiki/Optimization-killers#3-managing-arguments)中提到了一些安全使用`arguments`的小tips，我觉得很有意义。在使用`arguments`时我们应该**仅仅只**从下面这几个方面切入，

- 可以使用`arguments.length`
- 可以使用`arguments[i]`，但是`i`必须是数字，且不能超过边界
- 除了`arguments.length`和`arguments[i]`这两种形式外，不要以其他任何形式使用`arguments`
- 严格要求，`fn.apply(y, arguments)`（`Function#apply`这种形式都是ok的）是没问题的，但是其他的形式都是不行的，比如`.slice`



最后，其实我想说，关于这个`arguments`所谓的性能，可能在绝不多数场景下，都不会成为瓶颈。除非你的某一个功能模块或者业务模块承担着上亿级别的ops/s，否则这种微小的提升带来的收益基本上可以忽略不计。不过话说回来，这个座位业务知识的扩展到时个不错的门路。而且，说不准随着日后V8引擎的更新，它会越来越智能，自动将这些问题内置解决了呢？你说对吧？😂😂


# 参考

- [Benchmark: Array-ification of arguments](http://blog.izs.me/post/7746314700/benchmark-array-ification-of-arguments)
- [node-thunkify/pull/12](https://github.com/tj/node-thunkify/pull/12)
- [stackoverflow question](http://stackoverflow.com/questions/23509312/does-node-js-really-not-optimize-calls-to-slice-callarguments)




