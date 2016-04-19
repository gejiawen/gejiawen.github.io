postid: "deep-into-javascript-exec-function-expression"
title: 深入理解JavaScript系列（4）-立即调用的函数表达式
date: 2014-12-15 20:46:20
categories: [Javascript]
tags: [深入理解JavaScript系列]
---

本文是[深入理解JavaScript系列](http://blog.gejiawen.com/2014/11/13/deep-into-javascript-series/)的第**四**篇读文笔记，博客原文在[这里](http://www.cnblogs.com/TomXu/archive/2011/12/31/2289423.html)。

# 内容简要

本文阐述的内容是JavaScript中经常遇到的两个知识点：**自执行函数**和**函数闭包**。如果你之前稍微接触过过JavaScript，你应该能够明白我所指的意思，这里我就不像大叔原文中那么较真这个行为的具体叫法了。

在JavaScript的世界中，如果你能够对自执行函数和函数闭包了若指掌，在实际编码中能够信手拈来，那么，一般来说你JavaScript的功力至少有中级以上了，呵呵，这可能还是一种保守的估计。

如果你有读过流行JavaScript类库源码的话，你可能会发现，源码的作者对自执行函数和函数闭包的使用是比较频繁的，再结合一些具体的业务场景，往往会得到一些非常美妙的设计。如果你去悉心品读，可能会发现高手写出来的JavaScript代码和新手写出的JavaScript代码完全是天壤之别。

# BACKBONE

大叔的原文中只针对**自执行函数**作了比较透彻的说明，而对**函数闭包**仅仅用了一个示例就一笔带过。这篇读文笔记中，我将会针对这两点分别作一些详细说明，尽量用简明的话将我理解中的这两个概念阐述清楚。

## 自执行函数

### 什么是自执行？

首先，什么叫自执行？在JavaScript中函数在执行的时候会创建一个叫做**执行上下文**的东西，这里问题又来了，这个**执行上下文**又是什么东西呢？

简单的说，执行上下文就是**JavaScript代码在执行时创建的一个容器，这个容器中可以随时创建只属于这一块代码的变量，函数声明等等。**

其实，执行上下文是ECMA-262规定的一个非常抽象的概念，我们这里只要对这个概念有个把握就可以了，更多的解释我就不多作笔墨了，如有兴趣，可查阅相关文档。

上面说到，执行上下文其实代码执行时才会生成的一个东西，如果我就简单的写一段JavaScript代码放在这里，我并没有在浏览器中引入这个JavaScript片段执行它，那么它就不会有执行上下文了。所以，**自执行**的含义，简单来说，**一段JavaScript代码中自己执行了。**这里的一段JavaScript代码一般都是指一个JavaScript函数，所以这里的自执行就是指函数调用。

我们来看个例子，

```javascript
function makeCounter() {
    // 只能在makeCounter内部访问i
    var i = 0;

    return function () {
        console.log(++i);
    };
}
// 注意，counter和counter2是不同的实例，分别有自己范围内的i。

var counter = makeCounter();
counter(); // logs: 1
counter(); // logs: 2

var counter2 = makeCounter();
counter2(); // logs: 1
counter2(); // logs: 2

alert(i); // 引用错误：i没有defind（因为i是存在于makeCounter内部）。
```

这里，我们每次调用函数`makeCounter()`时，其实都会生成一个独立的执行上下文。具体来看，`makeCounter`生成的执行上下文中包含了一个变量`i`以及一个匿名函数。

这里需要特别提出的一点是，每个独立的执行上下文，其中的变量都是相互独立的，即`counter`和`counter2`其实是不同的实例。

### 问题的核心

当你声明类似这样的函数，

```javascript
function foo() {
    // function body
}

var foo2 = function() {
    // function body
}
```

我们可以简单在函数名`foo`（或者变量名`foo2`）的后面加上`()`即可实现自执行。如下，

```javascript
foo();
foo2();
```

那是不是意为着我只要在函数的后面加上一对`()`就可以达到自执行的目的呢？我们看下面的代码，

```javascript
function() {
    return 'test';
}();

function foo() {
    return 'test2';
}();
```

遗憾的是，这两种方式，不管是在匿名函数后加`()`还是在普通的函数声明后加`()`都达不到让函数自执行的目的。这两种情况下，你都会得到一个报错。

上面提到的两种错误方式，其实出错的原理还不太一样，

- 前者是JavaScript在解析`function`关键字时，默认其是函数声明，函数声明要求必须有一个函数名。
- 后者是一个函数声明，函数声明后直接跟一个`()`，这个`()`其实是一个分组操作符，这里报错的原因是因为分组操作符需要一个表达式语句而不是一个声明语句。

### 自执行函数表达式

经过上面的说明，我们知道，不管是匿名函数（虽然这个匿名的声明也有问题）还是函数`foo`其实都只是函数声明，而这里的`()`是一个运算符，它要求前面的东西必须为（函数）表达式！

所以，我们只需要将`()`前面的内容变成函数表达式就行了。我们看下面的代码，

```javascript
// 下面2个括弧()都会立即执行
(function(){ /* code */ }());
(function(){ /* code */ })();

// 由于括弧()和JS的&&，异或，逗号等操作符是在函数表达式和函数声明上消除歧义的
// 所以一旦解析器知道其中一个已经是表达式了，其它的也都默认为表达式了

var i = function(){ /* code */ }();
true && function(){ /* code */ }();
0, function(){ /* code */ }();

// 如果你不在意返回值，或者不怕难以阅读
// 你甚至可以在function前面加一元操作符号

!function(){ /* code */ }();
~function(){ /* code */ }();
-function(){ /* code */ }();
+function(){ /* code */ }();

// 上面这种使用一元表达式这种方式其实是不太常见的
// 而且有时候肯定在一些场景下存在一些弊端，因为一元表达式会有一个不为undefined的返回值
// 要想返回值为undefined，那么最保险的就是使用void关键字

void function(){/* code */}();
```

一般常用的两种形式就是`(function(){}());`和`(function(){})();`，大叔的原文中说第一种是推荐的写法，但是不知道为什么现在很多人都是用的第二种~~

### 区别

原文中还提到了这个话题。额，其实是英文原文的作者提到的。其实在我看来，**自执行匿名函数**和**立即执行函数表达式**的区别基本上可以忽略，在实际的使用其实都是一回事，只不过两种形式的函数主体不太一致。如下代码，

```javascript
(function() {
    return '我是自执行匿名函数';
})()；

(function foo() {
    foo();
})();
```

好吧，我承认第二种其实是不太常见的。

### Module模式

想想前篇文章说的[Module模式](http://blog.gejiawen.com/2014/12/01/deep-into-javascript-module-model/)，我们常常使用Module模式配合自执行函数来封装一个工具。

下面是一个例子，

```javascript
// 创建一个立即调用的匿名函数表达式
// return一个变量，其中这个变量里包含你要暴露的东西
// 返回的这个变量将赋值给counter，而不是外面声明的function自身

var counter = (function () {
    var i = 0;

    return {
        get: function () {
            return i;
        },
        set: function (val) {
            i = val;
        },
        increment: function () {
            return ++i;
        }
    };
} ());

// counter是一个带有多个属性的对象，上面的代码对于属性的体现其实是方法

counter.get(); // 0
counter.set(3);
counter.increment(); // 4
counter.increment(); // 5

counter.i; // undefined 因为i不是返回对象的属性
i; // 引用错误: i 没有定义（因为i只存在于闭包）
```

当然这里还用到了闭包的概念。我自己就经常使用这种技巧来封装一些配置类或者工具类的东西。封装后，只要暴露一个对象就可以了，从而达到了对内部变量的隐藏。

## 函数闭包

### 什么叫闭包？

什么叫（函数）闭包呢？各种专业文献上对这个词的解释比较抽象，不是太好理解。我个人对闭包的理解就是：**闭包就是一个带有了父作用域相关变量的函数。**或者更加通俗一点就是：**闭包就是能够读取其他函数内部变量的函数。**

我想先谈谈JavaScript中为什么会有闭包这个东西。

我们知道在JavaScript中，函数第一等公民，函数的用途非常广泛，函数可以参数传入另一个函数，还可以返回值从一个函数中返回。我们看下面的代码，

```javascript
function fn1() {
    var a = 1;

    return function fn2() {
        return 1 + a;
    };
}

var foo = fn1(); // typeof foo === 'function'
foo(); // 2
```

这里`foo = fn1()`后，`foo`其实是一个函数引用，通俗点说，`foo`就是一个函数表达式。那么这个`foo`在执行的时候，它需要访问变量`a`，但是这个`a`并没有在`fn2`中定义，它是定义在`fn1`中的。所以`foo`（也就是`fn2`）在执行的过程中，会向其父作用域（即`fn1`所在的作用域）查找变量`a`。此时，`fn2`中就保持了一个对父作用域的引用。

类似这样的场景就是我们所说的（函数）闭包。其实闭包从某种意义上来说，就是将函数内部和函数外部连接起来的一座桥梁。

### 闭包的作用

闭包最大的作用有两个，

- 读取函数内部的变量
- 保持对变量的持续引用

我们来看下面的一个例子，

```javascript
var name = "The Window";
var object = {
    name : "My Object",
    getNameFunc : function(){
        return function(){
            return this.name;
        };
    }
};

alert(object.getNameFunc()()); // The Window
```

作一点改动后，

```javascript
var name = "The Window";
var object = {
    name : "My Object",
    getNameFunc : function(){
        var self = this;
        return function(){
            return self.name;
        };
    }
};

alert(object.getNameFunc()()); // My Object
```

我们来稍微分析一下。第一种情况中，

`object.getNameFunc()`的执行结果是其实是一个函数引用。而且这个`getNameFunc`函数在执行时，其内部的`this`指针是指向`object`的。接下来，`object.getNameFunc()()`其实等价于，

```javascript
var name = "The Window";
(function() {
    return this.name;
})();
```

这个代码片段在执行的时候，会检索`this`的值。这里，它最终检索的结果就是全局对象`window`，然后返回的结果就是`name = 'The Window'`。

而第二种情况中，我们使用变量`self`暂存了匿名函数（其实就是`getNameFunc`函数表达式）的`this`指针，而这个`this`指针在运行时的指向正是`object`。函数`getNameFunc`返回的匿名函数毫无疑问，它是一个闭包，而且它保持了对父作用域变量`self`的持续引用。

更多内容，推荐阅读阮一峰的[学习Javascript闭包（Closure）](http://www.ruanyifeng.com/blog/2009/08/learning_javascript_closures.html)。

### 常见误区

在使用闭包的时候，有一个常见的误区，我们看下面的代码，

```javascript
for (var i = 0; i < 10; i++) {
    setTimeout(function(){
        console.log(i);
    }, 1000);
}
```

这段代码的运行结果将会连续打印10个10。

你可能会问：啊？怎么会这样？不是说好的打印从0到9的序列么？

我们来稍微分析一下。

`for`循环中连续创建了10个延时函数，每个延时函数的函数体是打印迭代变量`i`。这里我们先忽略10个延时函数由于创建先后顺序以及CPU时间片造成误差。当10次循环结束后，肯定还是没有经过1000ms，不过此时由于迭代的结果，迭代变量`i`已经变成10了。接下里延时计时器结束，开始执行延时函数，函数中需要访问变量`i`，不幸的是，此时的`i`已经变成10了，所以打印出来的10个数据都是10。

那我们如何修改能够达到我们本来的目的呢？即按照迭代变量的顺序，依次打印出0-9呢？

```javascript
for (var i = 0; i < 10; i++) {
    setTimeout((function(index){
        return function() {
            console.log(index);
        };
    })(i), 1000);
}
```

代码中应该看的很清楚了，延时函数中使用了一个闭包，这个闭包保持了对父作用域中参考变量`index`的持续引用，而这个`index`是随着每次for循环实时传递进来的迭代变量。所以它将会打印出0-9。

# 总结

这篇对**自执行函数**（或者叫**立即调用匿名函数表达式**）以及**函数闭包**作了细致的阐述，基本上涵盖了这两个知识点所有的方方面面，更多的内容就是需要在实际编码中进行实战了。

我还是想强调那句话，只要对JavaScript中的这两个要点了若指掌，编码时能够做到信手拈来，那么假以时日必定能够成为JavaScript高手。

