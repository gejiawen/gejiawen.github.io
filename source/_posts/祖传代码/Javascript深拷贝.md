postid: "javascript-deep-copy"
title: 'Javascript深拷贝'
date: 2014-07-18 16:22:14
categories: [祖传代码]
tags: [javascript, 实践]
---

因为js中的对象可以**无限嵌套**，所以一般来说，给js的对象做**深度拷贝**不是一个好的主意。但是有时候，我们偏偏又有这种需求，下面简要介绍下如何在js中做深度拷贝。


看下面的代码，支持**数组**和**对象**的深拷贝，

```javascript
var deepClone = function(obj) {
    if (typeof (obj) !== 'object') {
        return obj;
    }
    var re = {};
    if (obj.constructor === Array) {
        re = [];
    }
    for ( var i in obj) {
        // 过滤从上级原形继承的属性
        if (obj.hasOwnProperty(i)) {
            re[i] = deepClone(obj[i]);
        }
    }
    return re;
}
```

上面是一种非常常规的深拷贝思路。除此之外，还有一种非常取巧的方式，

```javascript
var a = {
    // your property
};

var aStr =JSON.stringify(a);

var copy = JSON.parse(aStr);
```

当`a`是一个符合json格式的对象时，上面是没有问题。

但是当`a`中出现`Function`类型的属性，或者不是标准的json数据，或者出现如下的形式：

```javascript
var a = {
    b: a
};
```
这些情况时，就得不到期望的结果了。


