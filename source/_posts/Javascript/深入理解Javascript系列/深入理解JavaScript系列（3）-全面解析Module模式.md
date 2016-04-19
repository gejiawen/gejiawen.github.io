postid: "deep-into-javascript-module-model"
title: 深入理解JavaScript系列（3）-全面解析Module模式
date: 2014-12-01 15:54:58
categories: [Javascript]
tags: [深入理解JavaScript系列]
---

本文是[深入理解JavaScript系列](http://blog.gejiawen.com/2014/11/13/deep-into-javascript-series/)的第**三**篇读文笔记，博客原文在[这里](http://www.cnblogs.com/TomXu/archive/2011/12/30/2288372.html)。

# 内容简要

本文是整个深入理解JavaScript系列中第一篇稍微有点难啃的文章，特别是对新手来说。大叔这篇文章的写作借鉴了下面两篇文章，

- [JavaScript Module Pattern: In-Depth](http://www.adequatelygood.com/JavaScript-Module-Pattern-In-Depth.html)
- [A JavaScript Module Pattern](http://yuiblog.com/blog/2007/06/12/module-pattern/)

主要参考的是前者。阐述的内容就是**模块化模式**。

首先，什么叫**模块化模式**？它具有哪些特点？

模块化模式（或者说模块化编程）在JavaScript中是一个非常常见的编程技巧，它按照功能（或者业务）将JavaScript代码封装在一起组成一个模块，对外暴露特定的额方法和属性。一般来说，模块化模式具有如下几个特点，

- 模块化，可重用
- 将一系列的方法及属性封装在一起，与其他模块彼此独立，不污染全局作用域，松耦合。（大叔这里说的松耦合的意思，我猜测应该就是指模块的独立性）
- 暴露特定的接口（方法或者属性），模块的其他方法外部不可访问

# BACKBONE

文章的主体将分别介绍模块化模式的基本用法和高级用法。

## 基本用法

话不多说，老夫先把大叔的一段代码拉出来溜溜，

```javascript
var Calculator = function (eq) {
    /*
    这里可以声明私有成员
    */

    var eqCtl = document.getElementById(eq);

    // 暴露公开的成员
    return {
        add: function (x, y) {
            var val = x + y;
            eqCtl.innerHTML = val;
        }
    };
};
```

我们可以通过如下的方式来调用：

```javascript
var calculator = new Calculator('eq');
calculator.add(2, 2);
```

这里使用了`new Calculator()`把函数`Calculator`当作一个构造函数调用。有的同学可能会问，这里不使用`new Calculator()`，直接调用`Calculator()`函数好像也是可以的啊，那`new Calculator()`和不使用`new`到底啥区别啊？

首先我在[之前的文章](http://blog.gejiawen.com/2014/09/19/deep-into-new-object/)中有阐述过`new xxx()`这个操作的本质，不太清楚的可以移步看看。

在大叔举的这个例子中，使用`new`与不使用`new`都是没有错的，下面的`calculator.add`方法都是可以调用的。

不过这两者有自己适合的场景，

- 使用`new`一定会返回一个object，其内部的`this`指向object自身，且object中方法或者属性可以通过`this`相互访问。这种是使用JavaScript构建OOP编程的基础。
- 不使用`new`的含义就是简单的调用函数，其返回值根据函数的`return`语句来确定，且内部的`this`都指向全局作用域。这种一般用于命名空间的管理，独立模块的封装。

## 匿名闭包和全局变量

闭包是函数式编程语言具有的一种语法糖（暂且这么说吧，虽然不太准备）。JavaScript中对闭包的应用随处可见，匿名闭包更是让一切成为可能的基础，而这也是JavaScript最灵活的特性。

下面我们来创建一个最简单的闭包函数，函数内部的代码一直存在于闭包内，在整个运行周期内，该闭包都保证了内部的代码处于私有状态。

```javascript
(function () {
    // ...所有的变量和function都在这里声明，并且作用域也只能在这个匿名闭包里
    // ...但是这里的代码依然可以访问外部全局的对象
})();
```

可以看出匿名闭包其实就是一个**自执行函数**。关于自执行函数在后面的文章中还会更加深入的讨论。现在只需要知道自执行函数的本质是一个函数表达式，在运行时将会产生一个封闭作用域（或者说私有作用域），这个封闭作用域可以访问外部的全局变量，但是不能被外部访问。

这里可能会遇到一个问题，比如下面的代码，

```javascript
var name = 'a';

(function() {
    console.log(name); // a
    name = 'b'; // 这里不带var的赋值，将会将name隐式的提升为全局变量，且覆盖了闭包外部的name
    console.log(name); // b
})();

console.log(name); // b
```

可见，由于JavaScript中存在**隐式全局变量**这样一种东西，当在闭包中需要访问外部的全局变量时，万一操作不当，就会隐式的声明一个你不知道的全局变量。

现在流行的JavaScript库，比如JQuery，都采用这样一种方式来达到从闭包内部访问外部的全局变量的目的，

```javascript
(function (window, undefined) {
    // JQuery的源码其实就是一个大闭包！！
})(window);
```

可见，我们可以将全局变量当成一个参数传入到匿名函数然后使用，相比隐式全局变量，它又清晰又快。

有时候可能不仅仅要使用全局变量，而是也想声明全局变量，如何做呢？我们可以通过将匿名函数的返回值赋值给这个全局变量，代码如下，

```javascript
var blogModule = (function() {
    var my = {},
        privateName = "博客园";

    function privateAddTopic(data) {
        // 这里是内部处理代码
    }

    my.Name = privateName;
    my.AddTopic = function (data) {
        privateAddTopic(data);
    };

    return my;
})();
```

上面的代码声明了一个全局变量`blogModule`，并且带有2个可访问的属性：`blogModule.AddTopic`和`blogModule.Name`，除此之外，其它代码都在匿名函数的闭包里保持着私有状态。

## 高级用法

下面将会阐述几种稍微高级一点的用法，其实都是对上面所说的基本用法的扩展。

### 扩展

前面的基本用法可能不太适合大型的项目。因为大型的项目往往会有多人协作开发，每个人负责的模块不尽相同，这时候就需要把一个模块分割到不同的文件中去。

看下面的代码，

```javascript
var blogModule = (function(my) {
    my.AddPhoto = function () {
        //添加内部代码
    };
    return my;
})(blogModule);
```

发现了没有，我们将`blogModule`自身作为参数传递给自执行函数。这个自执行函数执行完毕后，`blogModule`上就会多处一个方法`AddPhoto`。试想一下，多人开发中每个人都采用这种模式，那么完全就可以彼此独立的给`blogModule`添加自己所需的方法和属性。

一般来说，我们还会做一些异常判断，因为多人协作的过程中，每个人都不知道自己拿到的`blogModule`究竟有哪些东西，甚至都不知道这个对象是否为`undefined`的，改进如下，

```javascript
var blogModule = (function(my) {
    my.AddPhoto = function () {
        //添加内部代码
    };
    return my;
})(blogModule || {}); // 这里是重点
```

如上面的代码，我们在传递参数的时候，其实传入的是一个判断表达式，这样就保证了在闭包内部`blogModule`必定是不为`undefined`，是不是很巧妙？ :)

然而，有的时候我们在扩展的时候需要对某一些方法进行重写，那么该怎么办呢？看如下的代码，

```javascript
var blogModule = (function (my) {
    var oldAddPhotoMethod = my.AddPhoto;

    my.AddPhoto = function () {
        // 重写方法，依然可通过oldAddPhotoMethod调用旧的方法
    };

    return my;
})(blogModule);
```

代码中，我们使用私有变量`oldAddPhotoMethod`缓存了原先的`AddPhoto`方法，然后重写了`AddPhoto`方法。通过这种方式，我们达到了重写的目的，当然如果你想在继续在内部使用原有的属性，你可以调用`oldAddPhotoMethod`来用。

### 克隆和继承

```javascript
var blogModule = (function (old) {
    var my = {},
        key;

    for (key in old) {
        if (old.hasOwnProperty(key)) {
            my[key] = old[key];
        }
    }

    var oldAddPhotoMethod = old.AddPhoto;
    my.AddPhoto = function () {
        // 克隆以后，进行了重写，当然也可以继续调用oldAddPhotoMethod
    };

    return my;
})(blogModule);
```

说实话，我有点没太看懂这部分的内容，代码中的`for in`循环其实是将`old`对象克隆了一份赋值给`my`对象。但是这个克隆过程中，`old`对象中的`object`以及`function`类型的数据其实并没有发生克隆，只是多了个`my`对象的相应引用而已。

我不太明白的是，这个跟JavaScript的模块化有什么关系呢？望大神解惑。

### 跨文件共享私有对象

说实话，这部分的我看了很久才弄明白，别看代码量就10来行，但是真的不是那么好理解的。（也可能是我比较菜。）

这里的需求是这样的，一个module分割到多个文件中，我们想要各个文件中的私有变量能够交叉访问，该怎么做呢？

```javascript
var blogModule = (function (my) {
    var _private = my._private = my._private || {},

        _seal = my._seal = my._seal || function () {
            delete my._private;
            delete my._seal;
            delete my._unseal;

        },

        _unseal = my._unseal = my._unseal || function () {
            my._private = _private;
            my._seal = _seal;
            my._unseal = _unseal;
        };

    return my;
})(blogModule || {});
```

任何文件都可以对他们的局部变量`_private`设置，并且此设置对其他的文件也立即生效。一旦这个模块加载结束，应用会调用`blogModule._seal()`进行“上锁”，这会阻止外部接入内部的`_private`。如果这个模块需要再次增生，应用的生命周期内，任何文件都可以调用`_unseal()`进行“开锁”，然后再加载新文件，加载新文件后又会初始化`_pravite`、`_seal`和`_unseal`，然后再次调用`_seal()`“上锁”。

这个过程很巧妙。其实我觉得，~~各个文件中的私有变量能够交叉访问~~这个需求简直就是个奇葩，为什么各个文件中的私有变量要交叉访问呢？这不是破坏了模块的独立性么？有人说其实这里的多文件其实仍然描述的是同一个模块，那我想说，这种情况下使用**子模块**应该是一种更优的选择。

### 子模块

子模块是最简单也是最经常使用的设计思路，

```javascript
blogModule.subModule = (function () {
    var my = {};
    // ...

    return my;
})();
```

其实我个人觉得，子模块+模块扩展就足够应付一般的模块化模式需求了。使用过多的高级用法，必定给代码增加复杂度，给日后的维护和升级肯定为增加难度。

# 总结

就我目前的经验来说，模块化模式经常用到的几种方法包括，**松耦合扩展**，**私有作用域**以及**子模块**。这几种就是最常用的方式。

不过现在业界出现了CMD、AMD等规范后，JavaScript代码的模块化管理更趋向于代码级别而不再是设计级别。所以本文中所描述的模块化模式，我猜测日后将会越来越淡化，越来越轻量化，比如用在配置中，工具方法的管理上等等。

