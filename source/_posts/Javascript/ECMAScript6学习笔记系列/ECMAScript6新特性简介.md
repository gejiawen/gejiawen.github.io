postid: "es6-new-feature"
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
| 箭头操作符 | Arrows | `v => console.log(v)` | 类似于部分强类型语言中的lambda表达式 |
| 类的支持 | Classes | - | 原生支持类，让javascript的OOP编码更加地道 |
| 增强的对象字面量 | enhanced object literals | - | 增强对象字面量 |
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

何为箭头操作符？其实箭头操作符其实就是使用`=>`语法来代替函数。如果你熟悉C#或者Java之类的强类型语音，你应该知道[lambda语法](http://baike.baidu.com/link?url=Fvtz0awy-RBkcTfG0DtYWeOZU56pP6fkYoMZRtRFEO6eOKpDzLmJGHkUfUxfeM6Nf0ZcMUPlCZfSX1IcXFbORq)，其实箭头操作符做的事情跟lambda表达式有异曲同工之妙。具体的用法看下面的例子，

```javascript
var arr = [1, 2, 3];

// 传统写法
arr.forEach(function (v) {
    console.log(v);
});

// 使用箭头操作符
arr.forEach( v => console.log(v));
```

在传统写法中，我们需要写一个匿名的函数，然后给这个函数传入参数，在函数体中对此参数做相关处理。而使用箭头操作符后，它简化了函数的编写（其实是不需要再写匿名函数了）。

箭头操作符的左侧可以接收一个简单的参数，也可以使用一个参数列表。右侧进行具体的操作或者是返回值。下面的示例展示了箭头操作符更一般的用法，

```javascript
// 在表达式中使用
var odds = evens.map(v => v + 1);
var nums = evens.map((v, i) => v + i);
var pairs = evens.map(v => ({even: v, odd: v + 1}))

// 在申明中使用
nums.forEach(v => {
    if (v % 5 === 0) fives.push(v);
});

// 使用this指针
var bob = {
    _name: "Bob",
    _friends: [],
    printFriends() {
        this._friends.forEach(f => consol.log(this._name + " knows " + f));
    }
}
```

# 类的原生支持

ES6中提供了类的原生支持，引入了`class`关键字，其实它是一种实现OOP编程思想的语法糖。它是基于原型链的。

其实javascript本身就是面向对象的语音，只不过没有Java之类的语言那么学院派，它更加灵活。ES6中提供的类其实就是对原型模型的包装。

ES6提供原生`class`支持后，类方面的操作，比如类的创建、继承等等更加直观了。并且父类方法的调用、实例化、静态方法和构造函数等OOP概念都更加形象化了。

我们来看一些示例，看在ES6中到底如何使用学院化的使用类，

```javascript
// 类的定义
class Animal {
    // ES6中的构造器，相当于构造函数
    constructor(name) {
        this.name = name;
    }

    // 实例方法
    sayName() {
        console.log('My Name is ' + this.name);
    }
}

// 类的继承
class Programmer extends Animal {
    constructor(name) {
        // 直接调用父类构造器进行初始化
        super(name);
    }

    // 子类自己的实例方法
    program() {
        console.log('I\'am coding...');
    }

    // 静态方法
    static LEVEL() {
        console.log('LEVEL BAD!');
    }
}

// 一些测试
var doggy=new Animal('doggy'),
larry=new Programmer('larry');
doggy.sayName(); // ‘My name is doggy’
larry.sayName(); // ‘My name is larry’
larry.program(); // ‘I'm coding...’
Programmer.LEVEL(); // ‘LEVEL BAD!’
```

可以看到，ES6中我们可以完全用Java中的思想来写创建类、继承类、实例方法、初始化等等。

# 对象字面量的增强

在ES6中，对象字面量被增强了，写法更贱geek，同时可以在定义对象的时候做的事情更多了。比如，

- 可以在对象字面量里面定义原型
- 定义方法可以不用`function`关键字了
- 更加方便的定义方法
- 直接调用原型链上层（父层）的方法

具体的我们看下面的示例代码，

```javascript
var human = {
    breathe() {
        console.log('breathing...');
    }
};

function sleep() {
    console.log('sleeping...');
}

var worker = {
    __proto__: human, // 设置原型，相当于继承human
    company: 'code',
    sleep, // 相当于 `sleep: sleep`
    work() {
        console.log('working...');
    }
};

human.breathe(); // `breathing...`
worker.sleep(); // `sleeping...`
worker.work(); // `working...`
```

# 字符串模板

ES6中提供原生的字符串模板支持。其语法很简单使用使用反引号**`**来创建字符串模板，字符串模板中可以使用${placeholder}来生成占位符。

如果你有使用后端模板的经验，比如`smarty`之类的，那么你将对此不会太陌生。让我们来看段示例，

```javascript
var num = Math.random();
console.log(`your random num is ${num}`);
```

字符串模板其实并不是很难的东西，相对比较容易理解。个人觉得其目前主要的使用场景就是改变了动态生成字符串的方式，不再像以前那样使用字符串拼接的方式。

# 解构赋值

如果你熟悉python等语言，你对解构赋值的概念应该很了解。解构的含义简单点就是自动解析数组或者对象中值，解析出来之后一次赋值给一系列的变量。

利用这个特性，我们可以让一个函数返回一个数组，然后利用解构赋值得到数组中的每一个元素。让我们来看一些例子。

```javascript
function getVal() {
    return [1, 2];
}

var [x, y] = getValue();
var [name, age] = ['larry', 26];

console.log('x: ' + x + ', y: ' + y); // x: 1, y: 2
console.log('name: ' + name + ', age: ' + age); // name: larry, age: 26

// 交换两个变量的值
var [a, b] = [1, 2];
[a, b] = [b, a];
console.log('a: ' + a + ', b: ' + b);
```

# 函数参数的扩展

ES6中对函数参数作了很大的扩展，包括

- 默认参数
- 不定参数
- 扩展参数

默认参数，即我们可以为函数的参数设置默认值了。可能你在某些js库的源码经常会看到类似下面这样的操作，

```javascript
function foo(name) {
    name = name || 'larry';
    console.log(name);
}
```

上面的**或操作**其实就是为了给参数`name`设置默认值。

而在ES6中，我们可以采用更加方便的方式，如下，

```javascript
function foo(name = 'larry') {
    console.log(name);
}

foo(); // 'larry'
foo('gejiawen'); // 'gejiawen'
```

不定参数的意思是，我们的函数接受的参数是不确定的，由具体的调用时决定到底会有几个参数。其实在ES6之前的规范中我们大可以使用`arguments`变量来达到不定参数的判断，但是无疑这一方案过于复杂。

不定参数的用法是这样的，我们在函数参数中使用`...x`这样的方式来代表所有的参数。其中`x`的作用就是指代所有的不定参数集合（其实它就是一个数组）。让我们来看个例子，

```javascript
function add(...x) {
    return x.reduce((m, n) => m + n);
}

console.log(add(1, 2, 3)); // 6
console.log(add(1, 2, 3, 4, 5)); // 15
```

关于拓展参数，其实拓展参数其实一种语法糖，它允许传递一个数组（或者类数组）作为函数的参数，而不需要`apply`操作，函数内部可以自动将其当前不定参数解析，然后可以获得每一个数组（类数组）元素作为实际参数。看下面的示例，

```javascript
function foo(x, y, z) {
    return x + y + z;
}

console.log(foo(...[1, 2, 3])); // 6
```

# 新增`let`和`const`关键字

ES6中新增了两个关键字，分别为`let`和`const`。

`let`用于声明对象，作用类似`var`，但是`let`声明的变量拥有更狭隘的作用域，脱离特定的作用域后变量的生命周期就终结了。这将有效的避免javascript中由于**隐式变量作用域提升**而导致的各种bug，甚至是内存泄露等等问题。

`const`用于声明常量。

让我们来看一些例子。

```javascript
for (let i = 0; i < 2; i++) {
    console.log(i); // 0, 1
}
console.log(i); // undefined, 严格模式下会报错
```

# `for...of`遍历

ES6中新增了一种遍历数组和对象的方法，叫做`for...of`。他与`for...in`用法相似。看下面的例子，

```javascript
var arr = [1, 2, 3];

for (let v of arr) {
    console.log(v); // 1,2,3
}

for (let i in arr) {
    console.log(arr[i]); // 1,2,3
}
```

从上面的示例代码中，我们可以看出，`for...of`遍历时提供的是value，而`for...in`遍历提供的是key。

# 迭代器和生成器

ES6中新增的迭代器和生成器的相关概念相对来说有点复杂，后面将会产出专门的文章针对这个topic进行阐述。

# Promise

Promises是处理异步操作的一种模式，之前在很多三方库中有实现，比如jQuery的`deferred`对象。当你发起一个异步请求，并绑定了`.when()`，`.done()`等事件处理程序时，其实就是在应用promise模式。下面是一段示例代码，

```javascript
//创建promise
var promise = new Promise(function(resolve, reject) {
    // 进行一些异步或耗时操作
    if ( /*如果成功 */ ) {
        resolve("Stuff worked!");
    } else {
        reject(Error("It broke"));
    }
});

//绑定处理程序
promise.then(function(result) {
	//promise成功的话会执行这里
    console.log(result); // "Stuff worked!"
}, function(err) {
	//promise失败会执行这里
    console.log(err); // Error: "It broke"
});
```

关于promise更多的详细内容，后面将会产出专门的文章针对这个topic进行阐述。

# Unicode的支持

ES6中对Unicode的支持更加完善了，可以使用使用正则表达式的`u`标记对unicode字符串进行匹配。

```javascript
// same as ES5.1
"𠮷".length == 2

// new RegExp behaviour, opt-in ‘u’
"𠮷".match(/./u)[0].length == 2

// new form
"\u{20BB7}"=="𠮷"=="\uD842\uDFB7"

// new String ops
"𠮷".codePointAt(0) == 0x20BB7

// for-of iterates code points
for(var c of "𠮷") {
  console.log(c);
}
```

# 模块化的原生支持

在ES6中，javascript从语言层面开始支持module机制了。在ES6之前，各种模块化JS代码的机制和规范早就已经大行其道，比如CommonJS、AMC、CMD等。不过这些都是一些第三方的规范并不是javascript官方的标准规范。

ES6中定义的module机制的用法如下，

```javascript
// point.js文件
module "point" {
    export class Point {
        constructor (x, y) {
            public x = x;
            public y = y;
        }
    }
}

// app.js文件
module point from "/point.js";
import Point from "point"

var origin = new Point(0, 0);
console.log(origin.x, origin.y); // 0, 0
```

其中`app.js`中的前两句的作用是声明导入模块，并且指定需要导入的接口。其实这两句可以合并成一句，`import Point from "/point.js"`或者是这样`import * as Point from "/point.js"`。不过，如果合并成一句，在使用导入模块中的暴露接口时，需要这么来使用，

```javascript
var origin = new Point.Point(0, 0);
```

# 新增四种数据结构

`Map`,`Set`, `WeakMap`, `WeakSet`这四个数据结构是ES6中新增的集合类型。塔恩提供了更加方便的获取属性值的方法，不用像以前一样用`hasOwnProperty`来检查某个属性是属于原型链上的还是当前对象的。同时，在进行属性值添加与获取时有专门的`get`，`set`魔术方法。

看下面的示例代码，

```javascript
// Sets
var s = new Set();
s.add("hello").add("goodbye").add("hello");
s.size === 2;
s.has("hello") === true;

// Maps
var m = new Map();
m.set("hello", 42);
m.set(s, 34);
m.get(s) == 34;
```

有时候我们会把对象作为另一个对象的键用来存放属性值，普通集合类型，比如一个简单的对象会阻止垃圾回收器对这些作为属性键存在的对象的回收，有造成内存泄漏的危险。而`WeakMap`,`WeakSet`则更加安全些，这些作为属性键的对象如果没有别的变量在引用它们，则会被回收释放掉，具体还看下面的例子。

```javascript
// Weak Maps
var wm = new WeakMap();
wm.set(s, { extra: 42 });
wm.size === undefined

// Weak Sets
var ws = new WeakSet();
ws.add({ data: 42 });//因为添加到ws的这个临时对象没有其他变量引用它，所以ws不会保存它的值，也就是说这次添加其实没有意思
```

# 新增内置对象的APIs

对`Math`，`Number`，`String`还有`Object`等添加了许多新的API。下面的示例代码对这些新的API进行了简单展示。

```javascript
Number.EPSILON;
Number.isInteger(Infinity); // false
Number.isNaN("NaN"); // false

Math.acosh(3); // 1.762747174039086
Math.hypot(3, 4); // 5
Math.imul(Math.pow(2, 32) - 1, Math.pow(2, 32) - 2); // 2

"abcde".includes("cd"); // true
"abc".repeat(3); // "abcabcabc"

Array.from(document.querySelectorAll('*')); // Returns a real Array
Array.of(1, 2, 3); // Similar to new Array(...), but without special one-arg behavior
[0, 0, 0].fill(7, 1); // [0,7,7]
[1, 2, 3].find(x => x == 3); // 3
[1, 2, 3].findIndex(x => x == 2); // 1
[1, 2, 3, 4, 5].copyWithin(3, 0); // [1, 2, 3, 1, 2]
["a", "b", "c"].entries(); // iterator [0, "a"], [1,"b"], [2,"c"]
["a", "b", "c"].keys(); // iterator 0, 1, 2
["a", "b", "c"].values(); // iterator "a", "b", "c"

Object.assign(Point, { origin: new Point(0,0) });
```

# Symbols

我们知道对象其实是键值对的集合，而键通常来说是字符串。在ES6中，除了字符串外，我们还可以用symbol这种值来做为对象的键。`Symbol`是一种基本类型，像数字，字符串还有布尔一样，**它不是一个对象**。

`Symbol`通过调用symbol函数产生，它接收一个可选的名字参数，该函数返回的symbol是**唯一的**。之后就可以用这个返回值做为对象的键了。`Symbol`还可以用来创建**私有属性**，外部无法直接访问由symbol做为键的属性值。

看下面的演示代码，

```javascript
(function() {

    // 创建symbol
    var key = Symbol("key");

    function MyClass(privateData) {
        this[key] = privateData;
    }

    MyClass.prototype = {
        doStuff: function() {
            ... this[key] ...
        }
    };
})();

var c = new MyClass("hello")
c["key"] === undefined // 无法访问该属性，因为是私有的
```



# 参考列表

- [ECMA-262 5.0](http://www.ecma-international.org/ecma-262/6.0/)
- [es6features](https://github.com/lukehoban/es6features)
- [ECMAScript 6入门](http://es6.ruanyifeng.com/)
- [ES6新特性概览](http://www.cnblogs.com/Wayou/p/es6_new_features.html)



