title: 'function closure'
date: 2014-07-18 16:35:07
categories: [祖传代码]
tags: [javascript]
---

闭包在js中是一个非常常见的语法糖。下面将会介绍如何实现一个函数闭包。

看下面的代码

```javascript
var closure = function() {
    var arg = arguments;
    return function() {
        arg[0].apply(arg[1], Array.prototype.slice.call(arg).slice(2));
    };
};
```

函数`closure`接受一组参数，
* 第一个参数是一个`Function`
* 第二个参数是一个**函数上下文**，
* 第三个以及后面的参数都是可选的，将会被apply到第一个参数（也是一个函数）中作为形式参数。

函数`closure`将会返回一个**function reference**， 即它的返回值仍然是一个`Function`。

我们可以将`closure`直接添加到`Function.prototype`，将原生对象扩展，这样所有的js函数都天生具有了`closure`方法。不过通常我们**不推荐直接扩展原生对象**。

另一种方式，我们可以借助[underscore](http://underscorejs.org/)的`mixin`方法，来扩展`underscore`的内置方法。如下：

```javascript
_.mixin({
    closure: closure
});
```

下面是一些测试代码：

```javascript
var foo = function(name) {
    console.log(this.name);
    console.log(name);
};

foo('Pajjket');

var foo2 = _.closure(foo, {
    name: 'Gee',
    sex: 'male',
    age: 25
}, 'Pajjket Gee');

foo2();
```

下面是测试的输出结果：

```shell
        // 在chrome环境中，这里是一个空字符串
Pajjket
Gee
Pajjket Gee
```

End. All rights reserved `@gejiawen`.
