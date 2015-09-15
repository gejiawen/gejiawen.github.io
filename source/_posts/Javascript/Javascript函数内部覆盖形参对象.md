postid: "javascript-override-formal-args"
title: Javascript函数内部覆盖形参对象
date: 2014-06-19 17:37:25
categories: [Javascript]
tags: [javascript]
---

有一个js的函数，其参数是一个`object`型的数据，如果在函数内部使用另外一个对象将其覆盖，会发生什么事情呢？


现在有一个Javascript函数，如下：

```javascript
var setName = function(arg) {
    arg.name = 'Wangcai';
    arg = new Object();
    arg.name = '9527';
};

var dog = new Object();
setName(dog);
console.log(dog.name);
```

结果：

```javascript
Wangcai
```

这是为什么呢？

因为在函数`setName`中，将`dog`作为实参传入，在函数内部修改`dog`的`name`属性，此时`dog`的值为`{name: "WangCai"}`，紧接着，又`new`了一个新的对象覆盖了形参`arg`，并且随后改写了`arg.name`。

**其实此时的`arg`与一开始传入的形参`arg`已经不是同一个对象了，前者为传入的形参，后者为函数内容的局部变量。**

引用《Javascript高级程序设计》书中的一句话：

> 当在函数内部重写引用对象`arg`时，这个变量引用的就是一个局部对象了。而这个局部对象会在函数执行完毕后立即销毁。


