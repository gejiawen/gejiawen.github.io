postid: 'think-about-javascript-sort'
title: 关于Javascript中sort方法的联想
categories: [踩坑录]
tags: [javascript]
date: 2015-11-26 16:10:42

---

# 掉坑

近两天深陷需求快速迭代的漩涡之中，无瑕去写一些东西。今下午好不容易挤出点时间，随便写点东西，是关于前不久刚踩过的一个坑。

其实说起来也没有什么大不了的事情，就是没有掌握好Javascript中`sort`的用法，所以在一个具体的使用场景下踩了个坑。

简单来说，`sort()`方式是Javascript内置对象Array的一个实例方法，用于给数组元素进行排序的。

所谓实例方法，就意味着它可以像下面这样去使用，

```javascript
var arr = [1, 3, 13, 29, 2, 74, 24, 47, 172, 371, 32, 48];
var sorted = arr.sort();
```

注意，示例代码中，我们是`arr.sort()`而不是`Array.sort(arr)`去使用的。

这里涉及到**对象上的方法**以及**对象原型链上的方法**这两种方法的差异，这里我就不多费唇舌解释了，若有兴趣可移步这篇文章[浅谈Javascript中的原型继承](http://blog.gejiawen.com/2014/10/16/prototype-inherit-in-javascript/)。

言归正传，在上面的示例中，我们来输出`arr`和`sorted`这两个变量看一下他们的结果，如下图

![](/res/think-about-javascript-sort/001.png)

图中有两个要点，

- `sort()`方法的排序策略默认是按照字母表升序来排序的
- `sort()`直接在原数组上进行排序，而不是创建一个副本


所谓**字母表顺序**的含义就是按照字符编码的顺序进行排序。要实现这一点，首先应把数组的元素都转换成字符串（如有必要），然后再进行比较，进行比较。

所以你会看到图片中**172**排在了**2**的前面，因为字符串`"172"`比字符串`"2"`的第一个字符的ASCII编码要靠前。

# 问题

由上面我知道了，`sort()`方法其实是带有一个默认的排序策略的。如果我现在想换一种排序策略，那该如何做呢？

举个例子来说，我现在对一个数组（已经确定这个数组元素都是数字）按照自然顺序进行降序排序操作。

其实很简单，我们可以给`sort()`方法传递一个函数回调即可。例如，

```javascript
arr.sort(function (a, b) {
    return b - a;
});
```

得到的结果如下，

```bash
sorted:  [371, 172, 74, 48, 47, 32, 29, 24, 13, 3, 2, 1]
```

这里注意，我们给`sort()`传入了一个函数，此函数接受两个参数a和b，其返回值的规则如下，

- 若返回一个小于0的值，则说明a小于b。
- 若返回0，则说明a等于b。
- 若返回一个大于的值，则说明a大于b。


关于`sort()`更多的官方内容，请看[这里](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/sort)。

# 联想

我们知道函数在Javascript占有非常重要的地位，可以说函数式Javascript中的第一等公民。这同时也为Javascript的函数式编程提供了基础。

Javascript中的函数式编程主要体现在哪里呢？为何本是一门玩票性质的脚本语言现在却有如此大的魅力呢？

请允许我举个例子来说明一下。

在Java语言中，也有一个`sort()`方法，而且它的作用也是对数组元组进行排序。其一般用法大概是下面这个样子的，

```java
Integer[] arr = {213, 16, 2058, 54, 10, 1965, 57, 9};
Arrays.sort(arr);
System.out.print(arr);
```

而下面是Javascript版本的代码，

```javascript
var arr = [213, 16, 2058, 54, 10, 1965, 57, 9];
arr.sort();
console.log(arr);
```

上面的做法没有什么问题，而且也能得到我们想要的结果。

但是现在，假如我需要改变数组的排序策略。那么针对这两种语言版本的实现，分别该如何改进呢？

我们知道，Java是一门非常严肃的面向对象语言，它的语法中，方法（函数）其实跟一般的数据类型是一个level上的，而且方法本身不能作为参数传递给其他方法。在Java中，我们一般会对`Arrays.sort()`做一次重载，接受一个包含比较策略的对象（注意，这里仍然是接受对象，而不是函数方法）。

然后，你传入这个包含比较策略的对象，可能会是一个接口的实例。大概的代码应该如下，

```java
// 声明接口
public interface CompareStrategy<T> {
    int compare(T t, T t1);
    boolean equals(object o);
}

/**
 * 创建一个匿名接口示例，然后实现接口中的compare方法
 */
Arrays.sort(values, new CompareStrategy<Integer>() {
    public int compare(Integer v1, Integer v2) {
        return v2 - v1;
    }
});
```

这段代码虽然就几行，而且看起来也不是很复杂。但是，它涉及到了非常重要的语法知识（接口声明、接口实例化等）。这从软件工程的角度来说，其实是加大了复杂度，并不是十分可取。

而Javascript版本的实现就简单多了，如下

```javascript
values.sort(function (v1, v2) { return v2 - v1; });
```

没有接口，没有额外的参数对象，没有冗余代码，一行就搞定。


Javascript中的函数式特性允许我们像使用其他基本数据类型一样去使用函数。可以将其当做另一个函数参数，也可以作为另一个函数的返回值等等。而在Java等（非函数式）语言中，这并不是能简单就能实现的。














