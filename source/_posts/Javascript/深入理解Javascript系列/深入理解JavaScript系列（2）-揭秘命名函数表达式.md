postid: "deep-into-javascript-function-expression"
title: 深入理解JavaScript系列（2）-揭秘命名函数表达式
date: 2014-11-25 22:59:34
categories: [Javascript]
tags: [深入理解JavaScript系列]
---

本文是[深入理解JavaScript系列](http://blog.gejiawen.com/2014/11/13/deep-into-javascript-series/)的第**二**篇读文笔记，博客原文在[这里](http://www.cnblogs.com/TomXu/archive/2011/12/29/2290308.html)。

# 内容简要

本文阐述了JavaScript中关于函数的两个非常让人混淆的东西，分别是**函数表达式**和**函数声明**。

[原文](http://www.cnblogs.com/TomXu/archive/2011/12/29/2290308.html)可能由于写作的时间（2011-12-29）相距现在比较久远，在那之前JScript与JavaScript正在争夺浏览器脚本语言的霸主地位，所以大叔的文章中还提到了一些JScript的内容以及[SpiderMonkey](https://developer.mozilla.org/en-US/docs/Mozilla/Projects/SpiderMonkey)的东西（谁让那时候V8引擎还没出来呢）。

我的读文笔记中将会舍弃这些“糟粕”，说它们是糟粕并没有贬低的意思，只是相对现在来说，它们已经不太合适拿出来说了，没什么意义，都是历史的产物，它们存在的唯一价值就是推动了标准的发展和统一。我是标准的鉴定拥护者。（语言有点激进，不喜轻喷）

所以，本篇读文笔记的目的就是阐述清楚**函数表达式**和**函数声明**这两个概念的含义以及一些常规误区。

# BACKBONE

## 函数表达式和函数声明

那么，函数表达式和函数声明究竟是啥玩意呢？

在ECMAScript中，创建函数的最常用的两个方法就是函数表达式和函数声明（这里不讨论`new Function()`这种形式）。这两者之间的区别不是很严谨，因为ECMAScript中仅仅定义了：**函数声明必须带有标识符（Identifier，其实就是函数名），而函数表达式则可以省略这个标识符**。

所以我们得出下面的语法，

**函数声明**语法，

```javascript
function function_name(args1, arg2, arg3) {
    // function body
}
```

这里的`function_name`显然是不能忽略的，就是说**在函数声明中，你必须给函数起一个名字**，当然函数参数及其个数都是可选的。

**函数表达式**语法，

```javascript
function [function_name](args1, arg2, arg3) {
    // function body
}
```

这里`[function_name]`的意思是表示`function_name`是可以被忽略的。


由此可见，如果不声明函数名称，它肯定是表达式，可如果声明了函数名称的话，如何判断是函数声明还是函数表达式呢？

> ECMAScript是通过上下文来区分的，如果`function foo(){}`是作为赋值表达式的一部分的话，那它就是一个函数表达式，如果`function foo(){}`被包含在一个函数体内，或者位于程序的最顶部的话，那它就是一个函数声明。

我们下面来看个例子来理解一下上面的解释，

```javascript
function foo(){} // 函数声明，因为它是程序的一部分

var bar = function foo(){}; // 函数表达式，因为它是赋值表达式的一部分

new function bar(){}; // 函数表达式，因为它是new表达式

(function(){
    function bar(){} // 函数声明，因为它是函数体的一部分
})();
```

还有一种常见的表达式，大家可能见过的，如下

```javascript
(function foo() {
    // foo body
})();
```

在后面的文章中我们将会专门对这种语法进行说明。这就是在JavaScript中使用的比较广泛的**自执行函数**，或者叫**立即执行函数**。这里我们暂时不讨论自执行函数，我们看这个函数由两个`()`组成，第一个`()`中其实一个函数，那么这里的`foo`是函数声明还是函数表达式呢？

这里的`foo`函数是**函数表达式**。虽然这个函数有个名字`foo`，但是它是函数表达式。为什么呢？

因为这个`foo`函数是被`()`包裹起来的，而这个`()`在JavaScript中其实是一个操作符，叫做**分组操作符**，而且这个分组操作符内部只能包含表达式。我们看几个例子，

```javascript
function foo(){} // 函数声明

(function foo(){}); // 函数表达式：包含在分组操作符内

try {
    (var x = 5); // 报错，因为分组操作符只能包含表达式而不能包含语句，这里的var就是语句
} catch(err) {
    // SyntaxError
}
```

你在chrome浏览器上可以快速体验下。打开F12，在console中输入`{'x': 10}`，然后回车chrome为告诉你

```shell
SyntaxError: Unexpected token :
```

如果我们这么输入，`({'x': 10})`，回车之后你会得到一个Object。怎么样，很神奇吧。

## 函数表达式和函数声明的区别

函数表达式和函数声明存在着十分微妙的差别。

**函数声明会在任何表达式被解析和求值之前先被解析和求值**，即使你的声明在代码的最后一行，它也会在同作用域内第一个表达式之前被解析或者求值。这其实跟[深入理解JavaScript系列（1）-编写高质量JavaScript代码的基本要点](http://blog.gejiawen.com/2014/11/13/deep-into-javascript-points-about-good-javascript-code/#预解析)是很相似的。

来看个简单的例子，

```javascript
alert(fn()); // I am fn!
alert(vfoo); // undefined
alert(foo()); // 报错：foo is not defined

var vfoo = function foo() {
    return "I am foo!";
};

function fn() {
    return 'I am fn!';
}
```

这个例子中，我在代码的注释中给出了相应的结果。不知道你有没有答对呢。

这个例子虽然简单，但是有如下几点需要说明，

- 虽然`fn`函数是在`alert`之后声明的，但是它是函数声明，会优先所有的表达式和语句解析和执行，所以在`alert`语句执行的时候`fn`函数其实已经有定义了。所以执行`fn`函数后顺利的弹出了字符串。
- 第二个弹出`undefined`是因为`vfoo`变量是赋值之前就使用了，其值当然是`undefined`。
- 第三个可能有人不太理解，代码中明明声明函数`foo`啊，咋还提示我foo没有定义呢？

我这里解释一下第三点，JavaScript代码是顺序执行的，也就是说，前面的代码肯定是优先后面的代码执行，有人会说，你这不是在打自己的脸么？之前还说JavaScript中会有预解析这种事情呢。其实预解析（pre-parse）和执行时（runtime）是两回事。一段JavaScript代码在运行的时候，是先经历预解析过程，这个过程JavaScript引擎会做一些变量和函数的命名，全局变量和局部变量语义表的构建等等事情；然后就是正式的执行过程。

JavaScript在运行时是顺序的。结合我们这个例子，JavaScript引擎在预解析阶段，就会解析出来`vfoo`这个变量，此时这个`vfoo`变量就是**声明但是未赋值**的状态，其值在JavaScript中就是`undefined`。而且这个`function foo`正是我们所说的**函数表达式**语法，他在预解析阶段是不会做任何事情的。在正式的运行时阶段，JavaScript代码是从上到下，一句一句的执行的。很明显这里的`alert(foo())`语句在下面的赋值表达式之前，在执行`alert`时，还没有进行赋值操作呢，而函数`foo`的定义其实是在赋值的时候才有定义的，所以这里会报错，告诉你foo还没有定义。

## 常见用法和误区

ECMAScript中规定，函数表达式中函数的标识符（就是函数的名字）是可以忽略的。那么，我到底是用函数声明好呢，还是使用函数表达式好呢？如果使用了函数表达式，那我到底是省略函数名好呢，还是不省略函数名好呢？

使用函数声明还是使用函数表达式是视情况而定的，至于函数表达式需不需要省略标识符，一般的，如果你使用了函数表达式，我们是推荐你省略函数名的。但是在某些情况下，是推荐不省略函数名的。别晕，看我细细道来。

先来看一个例子，

```javascript
if (true) {
    function foo() {
        return 'first';
    }
} else {
    function foo() {
        return 'second';
    }
}

foo();
```

这个`foo`执行后到底会返回啥呢？有人说，肯定是`first`啊。其实是不一定的。函数声明在条件语句内虽然可以用，但是并没有被标准化，也就是说不同的环境可能有不同的执行结果。所以说我们将尽量避免这种情况。这种情况下，我们就应该使用函数表达式了。如下，

```javascript
var foo;
if (true) {
    foo = function() {
        return 'first';
    };
} else {
    foo = function() {
        return 'second';
    };
}

foo(); // first
```

我们再来看一个例子，

```javascript
var f = function foo(){
    return typeof foo; // foo是在内部作用域内有效
};

// foo在外部用于是不可见的
typeof foo; // "undefined"
f(); // "function"
```

看到这个例子后，大家应该很清楚了。函数表达式如果带了函数名，那么这个函数名只在函数的内部是可用的，在函数外部是未定义。这一点可以说是个小坑吧，注意下就行了。

另外一个值得一提是，给函数表达式添加函数名，可以在调试的时候对引擎更加友好，可以在调试堆栈上看到函数表达式的名字。这点大家知道就行，现代的JavaScript引擎都非常聪明，各种调试方法层出不穷，不必纠结这些黑暗技巧。

# 总结

这篇读文笔记由于舍弃了一部分JScript的内容，显得比较单薄，不过没关系，这篇笔记的主要目的就是为了阐述清楚函数声明和函数表达式的爱恨情仇。我想我应该把这两者的方方面面都应该说清楚了吧。而且函数在JavaScript中是第一等公民，其内容远不止函数声明和函数表达式这一点内容，比如各种高阶函数，匿名函数，多层闭包等等。要想成为一名合格的JavaScript程序员，那么JavaScript函数一定要信手拈来，这应该是JavaScript中比较考验功力的一个内容了吧。

