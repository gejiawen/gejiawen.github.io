postid: "es5-new-api"
title: "ECMAScript5.1及其新增API简介"
date: 2015-07-27 16:03:09
categories: [Javascript]
tags: [es5]

---

以前一种对ECMAScript的概念比较模糊，以及它跟Javascript、Jscript等脚本语言之间的关系不是太明白。本篇文章将就这些问题展示叙述。

# 什么是ECMAScript

业界所说的ECMAScript其实是指一种规范，或者说是一个标准。具体点来说，它其实就是一份文档。其官网地址是[http://www.ecmascript.org/](http://www.ecmascript.org/)。而我们通过所说的ECMAScript标准具体是指**ECMA-262**这份官方文档。

目前ECMAScript释出的最新版本是[Fifth Edition of ECMA-262](http://www.ecma-international.org/publications/files/ECMA-ST/ECMA-262.pdf)，不过本篇文章我们将暂时不会讲ES6的相关东西。

这里有一份ES5的[中文资料](http://yanhaijing.com/es5/)，可以认为其实官方文档的翻译版。


## 标准的实现

前面我们知道ECMAScript其实就是一份标准文档，有ECMA标准化组织的TC39委员会制定的。但是这个组织只负责制定标准并不负责实现标准。那么是谁来实现这份标准呢？回答是各大浏览器厂商。

目前市面上常见的实现ECMAScript标准的脚本语言包括Javascript、Jscript等。同时这些标准实现由于一些历史原因和客观原因，各个版本之间、各个浏览器厂商的实现有一些差异，甚至是很大的差异。

关于ECMAScript、Javascript、Jscript这三者之间的关系，可以参阅[三国志的故事(ECMAScript、JavaScript和JScript)](http://blog.csdn.net/xujiaxuliang/article/details/5963331)。


# ECMAScript 5.1

ECMAScript 5.1就是我们常说的es5。它在2012年就已经被公开。时至今日，除了一些较低版本的浏览器，各大主流浏览器都已经实现支持了es5的绝大部分特性。

这里有一份不错的关于es5的文档，并附带了许多有价值的注释，有兴趣可移步[Annotated ECMAScript 5.1](http://es5.github.io/)。

## 浏览器支持

一般来说，除了针对个别特性的特殊说明，各大主流浏览器都支持es5，包括

- Chrome 13+
- Firefox 4+
- Safari 5.1*
- IE 9*

其中IE9不支持es的严格模式，从IE10开始支持。Safari 5.1不支持`Function.prototype.bind`。

# ES5新特性简介

下面我们将会针对ES5新增的特性或者API作一些简要介绍。不过不会非常具体的讲解，因为可能部分新增特性的讲解可能都需要一篇文章的篇幅来描述。

## Strict Mode

**Strict Mode**, 即所谓的严格模式。严格模式的意义是为了提供一种更佳良好错误检查机制，让你规避掉一些语言本身的bad point。

比如在严格模式下，我们不可以使用一个未经声明的变量。看下面的示例代码，

```javascript
"use strict"
function foo() {
    var testVar = 4;
    return testVar;
}
// This causes a syntax error.
testVar = 5;
```

上面的示例代码中，我们试图对一个未定义的变量进行赋值操作。在一般情况下（非严格模式）Javascript的执行引擎会认为这个`testVar`是一个全局变量，这一隐式转换在某些时候将会带来各种bug。而在严格模式下，你将会直接得到一个语法错误提示。

开启严格模式的方法很简单，只需要在文件的顶部写上字符串`use strict`即可。当然这需要执行环境支持严格模式。另外由于`use strict`其实是一个字符串常量。那么即使遇到不支持严格模式的环境，这行字符串只会被安全的忽略，不会带来任何的问题。

关于严格模式的更多内容，可移步阅读这篇文章[Strict Mode (JavaScript)](https://msdn.microsoft.com/en-us/library/br230269.aspx)。

## JSON对象

ES5提供一个内置的（全局）JSON对象，可用来序列化（`JSON.stringfy`）和反序列化（`parse`）对象为JSON格式。其一般性用法如下，

```javascript
var test = {
    "name": "gejiawen",
    "age": 22
};
console.log(JSON.strinfy(test)); // '{"name": "gejiawen", "age": 22}'
console.log(JSON.parse('{"name": "larry"}')); // Object {name: "larry"}
```

JSON对象提供的`parse`方法还提供第二个参数，用于对具体的反序列化操作做处理。比如，

```javascript
JSON.parse('{"name": "gejiawen", "age": 22, "lucky": "13"}', function(key, value) {
    return typeof value === 'string' ? parseInt(value) : value;
});
```

这里我们在回调函数中对解析的每一对键值对作处理，如果其是一个数字字符串，我们则对其进行`parseInt`操作，确保返回的`value`必定是数值型的。

JSON对象提供的`stringify`方法也会提供第二个参数，用于解析的预处理，同时也会提供第三个参数，用于格式化json字符串。比如，

```javascript
var o = {
    name: 'gejiawen',
    age: 22,
    lucky: '13'    
};
var ret = JSON.stringify(o, function(key, value) {
    return typeof value === 'string' ? undefined : value;
}, 4);
console.log(ret);
```

上面代码在输出时，得到的字符串将不会再呈现一行输出，而是正常的格式化输出，并采用4个space进行格式化。

另外，如果预处理函数的返回值为`undefined`，则此键值对将不会包含在最终的JSON字符串中。所以上面代码最终的结果是，

```
"{
    "age": 22
}"
```

## 新增`Object`接口

| 对象 | 构造器 | 说明 |
| :--- | :--- | :--- |
| `Object` | `getPrototypeOf` | 返回对象的原型 |
| `Object` | `getOwnPropertyDescriptor` | 返回对象自有属性的属性描述符 |
| `Object` | `getOwnPropertyNames` | 返回一个数组，包括对象所有自有属性名称集合（包括不可枚举的属性） |
| `Object` | `create` | 创建一个拥有置顶原型和若干个指定属性的对象 |
| `Object` | `defineProperty` | 给对象定义一个新属性，或者修改已有的属性，并返回 |
| `Object` | `defineProperties` | 在一个对象上添加或修改一个或者多个自有属性，并返回该对象 |
| `Object` | `seal` | 锁定对象。阻止修改现有属性的特性，并阻止添加新属性。但是可以修改已有属性的值。 |
| `Object` | `freeze` | 冻结对象，阻止对对象的一切操作。冻结对象将永远不可变。 |
| `Object` | `preventExtensions` | 让一个对象变的不可扩展，也就是永远不能再添加新的属性。 |
| `Object` | `isSealed` | 判断对象是否被锁定 |
| `Object` | `isFrozen` | 判断对象是否被冻结 |
| `Object` | `isExtensible` | 判断对象是否可以被扩展 |
| `Object` | `keys` | 返回一个由给定对象的所有可枚举自身属性的属性名组成的数组 |

这里不打算对每个新增接口作详细描述，不过比较常用的有如下几个，

- `Object.create`
- `Object.defineProperties`
- `Object.keys`

更多具体的用法请参阅Mozilla开发者文档。

## 新增`Array`接口

| 对象 | 构造器 | 说明 |
| :--- | :--- | :--- |
| `Array.prototype` | `indexOf` | 返回根据给定元素找到的第一个索引值，否则返回-1 |
| `Array.prototype` | `lastIndexOf` | 方法返回指定元素在数组中的最后一个的索引，如果不存在则返回 -1 |
| `Array.prototype` | `every` | 测试数组的所有元素是否都通过了指定函数的测试 |
| `Array.prototype` | `some` | 测试数组中的某些元素是否通过了指定函数的测试 |
| `Array.prototype` | `forEach` | 让数组的每一项都执行一次给定的函数 |
| `Array.prototype` | `map` | 返回一个由原数组中的每个元素调用一个指定方法后的返回值组成的新数组 |
| `Array.prototype` | `filter` | 利用所有通过指定函数测试的元素创建一个新的数组，并返回 |
| `Array.prototype` | `reduce` | 接收一个函数作为累加器，数组中的每个值（从左到右）开始缩减，最终为一个值 |
| `Array.prototype` | `reduceRight` | 接受一个函数作为累加器，让每个值（从右到左，亦即从尾到头）缩减为一个值 |

新增的数组接口中，基本都是比较有用的接口。需要注意的一点是，有的数组方法是不返回新数组的，有的接口是返回一个新数组，就是说使用这些新接口时，需要注意一下方法的返回值。

另外`Array`还有一个新增的接口，`Array.isArray`。显然此新增接口并不是实例方案，而是类似“静态方法”一样的存在。其作用很简单，就是判断某一对象是否为数组。

更多具体的用法请参阅Mozilla开发者文档。

## `Function.prototype.bind`

bind()方法会创建一个新函数,称为绑定函数.当调用这个绑定函数时,绑定函数会以创建它时传入 bind()方法的第一个参数作为 this,传入 bind() 方法的第二个以及以后的参数加上绑定函数运行时本身的参数按照顺序作为原函数的参数来调用原函数。

此方法的用法如下，

```javascript
fun.bind(thisArg[, arg1[, arg2[, ...]]])
```

使用`bind`可以为函数自定义`this`指针。它的常见使用场景如下，

```javascript
this.x = 9; 
var module = {
    x: 81,
    getX: function() {
        return this.x;
    }
};
module.getX(); // 81
var getX = module.getX;
getX(); // 9, 因为在这个例子中，"this"指向全局对象
// 创建一个'this'绑定到module的函数
var boundGetX = getX.bind(module);
boundGetX(); // 81
```

Javascript中重新绑定`this`变量的语法糖还有`call`和`apply`。不过`bind`显然与它们有着明显的不同。`bind`将会返回一个新的函数，而`call`或者`apply`并不会返回一个新的函数，它们将会使用新的`this`指针直接进行函数调用。

# 参考列表

- [https://developer.mozilla.org/zh-CN/docs/Web/JavaScript](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript)
- [翻译：ECMAScript 5.1简介](http://www.zhangxinxu.com/wordpress/2012/01/introducing-ecmascript-5-1/)

