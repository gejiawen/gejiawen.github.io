title: "ECMAScript6新特性简介"
date: 2015-07-28 18:26:10
categories: [Javascript]
tags: [es6, ECMAScript6学习笔记系列]

---

ES6（ECMAScript 6）终于在2015年6月正式发布了。距离上一次正式公开的ES5（于2009年发布的）已经相距了不短的时间了。其实在ES6正式发布之前的一段时间内，ECMA组织已经不再向ES6中添加新特性了，所以其实在ES6正式发布之前，业界已经有许多对ES6的相关实践了。比如阮一峰的[ECMAScript 6入门](http://es6.ruanyifeng.com/)其实就是早于ES6的正式发布时间的。

本文将主要基于[lukehoban/es6features](https://github.com/lukehoban/es6features)，参考众多的博客文章资料，对新出的ES6中新增的特性做一个简介。后续本系列将会产出针对ES6不同特性的文章。

# ES6新特性列表

下面的表格给出了ES6包含的所有特性，

| 新增特性 | 关键词 | 用法 | 描述 |
| :--- | :--- | :--- | :--- |
| 箭头操作符 | Arrows | `v = > console.log(v)` | 类似于部分强类型语言中的lambda表达式 |
| 类的支持 | Classes | - | 原生支持类，让javascript的OOP编码更加地道 |
| 增强的对象字面量 | enhanced object literals | - | 增强字面量的支持 |
| 字符串模板 | template strings | `${num}` | 原生支持字符串模板，不再需要第三方库的支持 |
| 解构赋值 | destructuring | `[x, y] = ['hello', 'world']` | 使用过python的话，你应该很熟悉这个语法 |
| 函数参数扩展 | default, rest, spread | - | 函数参数可以使用默认值、不定参数以及拓展参数了 |
| let、const | let、const | - | javascript中可以使用块级作用域和声明常量了 |
| for...of遍历 | for...of | `for (v of someArray) { ... }` | 又多了一种折腾数组、Map等数据结构的方法了 |
| 迭代器和生成器 | iterators, generator, iterables | - | ES6较为难以理解的新东西，后面会有相关文章 |
| Unicode | unicode | - | 原生的unicode更加完美的支持 |
| 模块和模块加载 | modules, modules loader | - | ES6中开始支持原生模块化啦 |
| map, set, weakmap, weakset | - | - | 新的数据结构 |
| 监控代理 | proxies | - | 我们可以监听对象发生了哪些事，并可以自定义对应的操作 |
| Symbols | - | - | 我们可以使用symbol来创建一个不同寻常的key |
| Promises | - | - | 这家伙经常在讨论异步处理流程时被提到 |
| 新的API | math, number, string, array, object | - | 原生的功能性API就是方便些 |
| 内置对象可以被继承 | subclassable built-ins | - | 可以基于内置对象，比如Array，来生成一个类 |
| 二进制、八进制字面量 | - | - | 可以直接在es6中使用二进制或者八进制字面量了 |
| Reflect API | - | - | 反射API？ |
| 尾调用 | tail calls | - | ES6中会自动帮你做一些尾递归方面的优化 |

ok，上面就是es6中涉及到的所有的新增特性。下面我们针对每一个topic，举一个sample example加以说明，不过并不会深入阐述。

# 箭头操作符

# 类的原生支持

# 对象字面量的增强

# 字符串模板

# 解构赋值

# 函数参数的扩展

# 新增`let`和`const`关键字

# `for...of`遍历

# 迭代器和生成器

# Promise

# Unicode的支持

# 模块化的原生支持

# 新增四种数据结构

# 新增内置对象的APIs


# 参考列表

- [ECMA-262 5.0](http://www.ecma-international.org/ecma-262/6.0/)
- [es6features](https://github.com/lukehoban/es6features)
- [ECMAScript 6入门](http://es6.ruanyifeng.com/)
- [ES6新特性概览](http://www.cnblogs.com/Wayou/p/es6_new_features.html)

End! All rights reserved `@gejiawen`.
