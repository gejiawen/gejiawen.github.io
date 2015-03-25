title: Javascript模块模式
date: 2014-10-14 17:46:11
categories: [大前端]
tags: [javascript, 翻译]
---

- 英文原文 [The JavaScript Module Pattern](http://javascriptplayground.com/blog/2012/04/javascript-module-pattern/)
- 译者 [gejiawen](https://github.com/gejiawen)

假设现在我们有个简单的js库，其目的是用来自增数字，

```javascript
var jspy = {
    count: 0,

    incrementCount: function() {
        this.count++;
    },

    decrementCount: function() {
        this.count--;
    },

    getCount: function() {
        return this.count;
    }
};
```

但是，使用者仍然可以通过`jspy.count = 5`这种方法来强制改变这个值。这种情况并不是我们最初的目的。在其他的语言中，你可以简单的定义一个**私有变量**来达到此目的，但是Javascript中不能定义**真正**的私有变量。不过，我们可以通过操作Javascript做一些额外的事情来实现。

这为我们带来了流行的Javascript设计模式，**模块模式**。

针对上述问题的解决方案如下，

```javascript
var jspy = (function() {
    var _count = 0;

    var incrementCount = function() {
        _count++;
    };

    var getCount = function() {
        return _count;
    };

    return {
        incrementCount: incrementCount,
        getCount: getCount
    };

})();
```

首先我们创造一个`_count`变量，下划线表明它是一个私有变量。不过在Javascript中下划线其实并没有什么实际的意义，但是它是一个用来标明私有变量的普遍用法。现在函数就可以操纵、返回变量了。

然而，你注意到了我把整个库包含在了一个自调用匿名函数中。这是一个在执行过程中马上被执行的函数。这个函数运行，定义了函数和变量然后到了`return {}`的部分，它告诉函数将其返回给变量`jspy`，或者换句话说，暴露给用户。我们暴露两个函数而不是`_count`变量，这意味着我们可以做如下操作，

```javascript
jspy.incrementCount();
jspy.getCount();
```

但是当我们试图进行如下操作时，

```javascript
jspy._count; //undefined
```

它将返回undefined。

对于上面的这种设计模式有许多不同的实现方法。有人喜欢在`return`中定义函数，

```javascript
var jspy = (function() {
    var _count = 0;

    return {
        incrementCount: function() {
            _count++;
        },
        getCount: function() {
            return _count;
        }
    };
})();
```

受到上面例子的启发，CHristian Heilmann提出了**Revealing Module Pattern**。

他的方法是将所有方法定义为私有变量，也就是说，不在`return`中定义，但是在那里暴露给用户，如下所示，

```javascript
var jspy = (function() {
    var _count = 0;

    var incrementCount = function() {
        _count++;
    };

    var resetCount = function() {
        _count = 0;
    };

    var getCount = function() {
        return _count;
    };

    return {
        add: incrementCount,
        reset: resetCount,
        get: getCount
    };
})();
```

这种设计模式有如下两个好处，

- 首先，它使我们更容易的了解暴露的函数。当你不在return中定义函数时，我们能轻松的了解到每一行就是一个暴露的函数，这时我们阅读代码更加轻松。
- 其次，你可以用简短的名字（例如 add）来暴露函数，但在定义的时候仍然可以使用冗余的定义方法（例如incrementCount）。

End. All rights reserved `@gejiawen`.
