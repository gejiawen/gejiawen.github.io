title: "JS中易混淆方法备忘录"
date: 2015-04-02 17:52:46
categories: [Javascript]
tags: [javascript]
---

> 本篇文章将会持续更新，以便收录将来可能会出现让博主犯糊涂的方法们。。。。

`Array`对象是Javascript内置的[对象](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array)之一，随着JavaScript版本标准的推进，内置对象开始支持越来越多的内置方法，以前需要借助第三方工具库，比如[underscore.js](http://underscorejs.org/)或者[Lo-dash.js](https://lodash.com/)，现在基本上内置对象已经提供了原生支持，唯一可能的瓶颈在于浏览器的支持广度。

本篇文章将会简述几组功能相似Javascript内置对象提供的方法的用法。当然这些内容去翻一下文档完全能够弄明白，此篇文章仅是博主自我的备忘，因为我在实际使用时，经常会遇到一种情况，就是会混淆两种相似的方法。

# `Array`对象

关于`Array`对象，一些非常常用的方法，比如`push()`或者`pop()`之类的，就没必要说了。这里，我将`Array`对象提供的方法分为两大类，一类是遍历类，一类是操作类。

遍历类的方法一般都会对数组进行遍历操作，比如`map()`之类的方法，而操作类基本上就是数组进行处理，可能会修改原数组，亦可能返回一个新数组。

## `slice()`和`splice()`

啊，这是我经常弄混淆的一组方法，经常会将它们做的事情弄反了。

`slice()`，可提取数组、字符串的某个部分，而且不修改原始对象，返回新对象。原型为，

```javascript
// 切片包含start但是不包含end下标
slice(start, end);
```

`splice()`，用于插入、删除、替换数组元素。原型为，

```javascript
// 数组从 start下标开始，删除deleteCount 个元素，并且可以在这个位置开始添加 n个元素
splice (start, deleteCount, [item1[, item2[, . . . [,itemN]]]]);
```

题外话，

经常在一些工具库中可能会看到下面这一段代码，

```javascript
Array.prototype.slice.call(arguments);
```

它的作用是这样的，利用`slice`方法，将**类数组**（Array-liked）对象转变成真正的数组。


## `forEach()`、`map()`、`filter()`

这些具有遍历属性的方法，有一个重要的区别标准，就是**每次迭代是否返回一个值，最终这些返回值组成一个新的数组。**

`forEach()`方法就是简单的遍历数组元素，而`map()`和`filter()`方法每次迭代将会返回一个值，然后最终根据返回的值组合生成一个新的数组。

## `shift()`和`unshift()`

我不得不吐槽一下这两个方法，因为我经常把它们的功能弄反了。

`shift()`，作用是移除原数组的**第一个**数组元素。其返回值就是那个被移除的元素。

而`unshift()`的作用恰好相反，它是往数组的头部添加元素，而且可以一次性添加多个，其原型如下

```javascript
arr.unshift([element1[, ...[, elementN]]])
```

其返回值为添加了新元素之后的数组长度。

需要注意的是，这两个方法都会直接对原数组进行修改。


# `Function`对象

`Function`对象中的方法不是很多，但是却非常有用。

## `call()`、`apply()`和`bind()`

这三个方法的作用类似，都可以重新绑定函数执行时的上下文（`this`变量的指向）。不过有些细节上的区别。

`call()`函数的原型如下，

```javascript
// 第一个参数为`this`变量的指向，后面为函数的实际参数
fun.call(thisArg[, arg1[, arg2[, ...]]])
```

`apply()`函数的原型如下，

```javascript
// 第一个参数为`this`变量的指向，函数的实际参数将会组合起来呈一个数组的形式进行传递
fun.apply(thisArg, [argsArray])
```

我们来看个实际的例子，

```javascript
var data = {
    name: 'erik',
    getName: function(funcName) {
        return this.name + ', funcName: ' + funcName;
    }
};

var self = {
    name: 'gejiawen'
};

console.log(data.getName('null')); // erik, funcName: null
console.log(data.getName.call(self, 'call')); // gejiawen, funcName: call
console.log(data.getName.apply(self, ['apply'])); // gejiawen, funcName: apply
```

说完了`call()`和`apply()`，我们再来说一下`bind()`。`bind()`也拥有和`call()`和`apply()`类似的功能，就是重新定义函数执行时的上下文。但是除此之外`bind()`还有另外一个功能，**它将返回一个函数引用（匿名函数）而不是立即执行函数。**

我们来看个例子，

```javascript
var name = 'gejiawen';

var data = {
    name: 'erik',
    getName: function() {
        return this.name;
    }
};

var gn = data.getName.bind(this);

console.log(typeof gn); // function
console.log(gn()); // gejiawen
console.log(data.getName.bind(data)()); // erik
```
# `String`对象

## `substr()`和`substring()`

这两个函数做的事情都是在一个字符串截取一段子串并返回，不过截取策略不太一致。

`substr()`函数的原型如下，

```javascript
// 给出一个开始位置和截取的长度，若不提供长度参数，则默认截取到原字符串的结尾
str.substr(start[, length])
```

`substring()`字符串的原型如下，

```javascript
// 给出一个开始下标和一个结束小标，截取开始和结束之间的长度，包括开始但是不包括结尾；若不提供结束参数，则默认截取到原字符串的结尾
str.substring(indexA[, indexB])
```

针对`substring()`函数还有个有趣的地方，如果结束参数大于开始参数，那么`substring()`内部会自动将两者交换，即`str.substring(1, 0) == str.substring(0, 1)`。



TBC......


End! All rights reserved `@gejiawen`.
