postid: "2-way-data-binding-and-define-property"
title: "JS中的双向数据绑定及Object.defineProperty方法"
date: 2015-04-02 10:35:01
categories: [Javascript]
tags: [javascript]
---

# 缘起

前几天在看一些流行的迷你mvvm框架（比如[avalon.js](http://avalonjs.github.io/)、[vue.js](http://vuejs.org/)这种较轻的框架，而非Angularjs、Emberjs这种较重的框架）的实现。现代流行的mvvm框架一般都会将数据双向绑定（two-ways data binding）做掉，作为框架自身的一个卖点（[Ember.js](http://emberjs.com/)貌似是不支持数据双向绑定的。），而且每种框架双向数据绑定的实现方式都不太一致，比如Anguarjs内部使用的是**脏检查**，而avalon.js内部实现方式的本质是设置**属性访问器**。

这里不打算具体的讨论各个框架对双向数据绑定的具体实现，仅说一下前端实现双向数据绑定的几种常用方法，并着重讲一下avalon.js实现双向数据绑定的技术选型。

# 双向数据绑定的常规实现方式

首先我们来说一下何为前端的**双向数据绑定**。简单的来说，就是框架的控制器层（这里的控制器层是一个泛指，可以理解为控制view行为和联系model层的中间件）和UI展示层（view层）建立一个双向的数据通道。当这两层中的任何一方发生变化时，另一层将会立即（或者看起来是**立即**）自动作出相应的变化。

一般来说要实现这种双向数据绑定关系（控制器层与展示层的关联过程），在前端目前会有三种方式，

1. 脏检查
2. 观察机制
3. 封装属性访问器

## 脏检查

我们说Angularjs（这里特指AngularJS 1.x.x版本，不代表AngularJS 2.x.x版本）双向数据绑定的技术实现是脏检查，大致的原理就是，Angularjs内部会维护一个序列，将所有需要监控的属性放在这个序列中，当发生某些特定事件时（注意，这里并不是定时的而是由某些特殊事件触发的），Angularjs会调用`$digest`方法，这个方法内部做的逻辑就是遍历所有的watcher，对被监控的属性做对比，对比其在方法调用前后属性值有没有发生变化，如果发生变化，则调用对应的handler。网上有许多剖析Angularjs双向数据绑定实现原理的文章，比如[这篇](http://www.html-js.com/article/2145)，再比如[这篇](http://www.zhihu.com/question/23275373)，等等。

这种方式的缺点很明显，遍历轮训watcher是非常消耗性能的，特别是当单页的监控数量达到一个数量级的时候。

## 观察机制

博主之前有一篇转载翻译的文章，[Object.observe()带来的数据绑定变革](http://gejiawen.github.io/2014/10/30/%E5%A4%A7%E5%89%8D%E7%AB%AF/Object-observe-%E5%B8%A6%E6%9D%A5%E7%9A%84%E6%95%B0%E6%8D%AE%E7%BB%91%E5%AE%9A%E5%8F%98%E9%9D%A9/)，说的就是使用ECMAScript7中的[`Object.observe`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/observe)方法对对象（或者其属性）进行监控观察，一旦其发生变化时，将会执行相应的handler。

这是目前监控属性数据变更最完美的一种方法，语言（浏览器）原生支持，没有什么比这个更好了。唯一的遗憾就是目前支持广度还不行，有待全面推广。

## 封装属性访问器

在php中有[魔术方法](http://php.net/manual/zh/language.oop5.magic.php)这样一种概念，比如php中的`__get()`和`__set()`方法。在javascript中也有类似的概念，不过不叫魔术方法，而是叫做访问器。我们来看个示例代码，

```javascript
var data = {
    name: "erik",
    getName: function() {
        return this.name;
    },
    setName: function(name) {
        this.name = name;
    }
};
```

从上面的代码中我们可以管中窥豹，比如`data`中的`getName()`和`setName()`方法，我们可以简单的将其看成`data.name`的访问器（或者叫做**存取器**）。

其实，针对上述的代码，更加严格一点的话，不允许直接访问`data.name`属性，所有对`data.name`的读写都必须通过`data.getName()`和`data.setName()`方法。所以，想象一下，一旦某个属性不允许对其进行直接读写，而必须是通过访问器进行读写时，那么我当然通过重写属性的访问器方法来做一些额外的情，比如属性值变更监控。使用属性访问器来做数据双向绑定的原理就是在此。

这种方法当然也有弊端，最突出的就是每添加一个属性监控，都必须为这个属性添加对应访问器方法，否则这个属性的变更就无法捕获。

# `Object.defineProperty`方法

国产mvvm框架avalon.js实现数据双向绑定的原理就是属性访问器。不过它当然不会像上述示例代码一样原始。它使用了ECMAScript5.1（ECMA-262）中定义的标准属性[`Object.defineProperty`](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/defineProperty)方法。针对国内行情，部分还不支持`Object.defineProperty`低级浏览器采用VBScript作了完美兼容，不像其他的mvvm框架已经逐渐放弃对低端浏览器的支持。

我们先来MDN上对`Object.defineProperty`方法的定义，

> The **Object.defineProperty()** method defines a new property directly on an object, or modifies an existing property on an object, and returns the object.

意义很明确，`Object.defineProperty`方法提供了一种直接的方式来定义对象属性或者修改已有对象属性。其方法原型如下，

```javascript
Object.defineProperty(obj, prop, descriptor)
```

其中，

- `obj`，待修改的对象
- `prop`，带修改的属性名称
- `descriptor`，待修改属性的相关描述

`descriptor`要求传入一个对象，其默认值如下，

```javascript
/**
 * @{param} descriptor
 */

{
    configurable: false,
    enumerable: false,
    writable: false,
    value: null,
    set: undefined,
    get: undefined
}
```

1. `configurable`，属性是否可配置。可配置的含义包括：是否可以删除属性（`delete`），是否可以修改属性的`writable`、`enumerable`、`configurable`属性。
2. `enumerable`，属性是否可枚举。可枚举的含义包括：是否可以通过`for...in`遍历到，是否可以通过`Object.keys()`方法获取属性名称。
3. `writable`，属性是否可重写。可重写的含义包括：是否可以对属性进行重新赋值。
4. `value`，属性的默认值。
5. `set`，属性的重写器（暂且这么叫）。一旦属性被重新赋值，此方法被自动调用。
6. `get`，属性的读取器（暂且这么叫）。一旦属性被访问读取，此方法被自动调用。

下面来一段示例代码，

```javascript
var o = {};

Object.defineProperty(o, 'name', {
    value: 'erik'
});

console.log(Object.getOwnPropertyDescriptor(o, 'name')); // Object {value: "erik", writable: false, enumerable: false, configurable: false}

Object.defineProperty(o, 'age', {
    value: 26,
    configurable: true,
    writable: true
});

console.log(o.age); // 26

o.age = 18;
console.log(o.age); // 18. 因为age属性是可重写的

console.log(Object.keys(o)); // []. name和age属性都不是可枚举的

Object.defineProperty(o, 'sex', {
    value: 'male',
    writable: false
});

o.sex = 'female'; // 这里的赋值其实是不起作用的
console.log(o.sex); // 'male';

delete o.sex; // false, 属性删除的动作也是无效的

```

经过上述的示例，正常情况下`Object.definePropert()`的使用都是比较简单的。

不过还是有一点需要额外注意一下，**`Object.defineProperty()`方法设置属性时，属性不能同时声明访问器属性（`set`和`get`）和`writable`或者`value`属性。**意思就是，某个属性设置了`writable`或者`value`属性，那么这个属性就不能声明`get`和`set`了，反之亦然。

因为`Object.defineProperty()`在声明一个属性时，不允许同一个属性出现两种以上存取访问控制。

示例代码，

```javascript
var o = {},
    myName = 'erik';

Object.defineProperty(o, 'name', {
    value: myName,
    set: function(name) {
        myName = name;
    },
    get: function() {
        return myName;
    }
});
```

上面的代码看起来貌似是没有什么问题，但是真正执行时会报错，报错如下，

```bash
TypeError: Invalid property.  A property cannot both have accessors and be writable or have a value, #<Object>
```

因为这里的`name`属性同时声明了`value`特性和`set`及`get`特性，这两者提供了两种对`name`属性的读写控制。这里如果不声明`value`特性，而是声明`writable`特性，结果也是一样的，同样会报错。

