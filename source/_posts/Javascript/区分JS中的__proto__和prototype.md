postid: "different-from-proto-and-prototype"
title: 区分JS中的__proto__和prototype
date: 2015-03-18 16:29:26
categories: [Javascript]
tags: [javascript]
---

我们知道javascript是**基于原型（prototype）继承**的语言。关于javascript原型继承的更多内容请参阅之前的这篇文章[浅谈Javascript中的原型继承](http://blog.gejiawen.com/2014/10/16/prototype-inherit-in-javascript/)，内容绝对不会让你失望。

不过本篇文章将会详细阐述javascript中用于原型继承的两个重要的东西`__proto__`和`prototype`，以及由它们延伸出的一点容易让人疑惑的方面。

# `__proto__`和`prototype`的概念

**`prototype`**就是原型的意思，就是函数（构造器）的一个属性，每个函数都将会有一个`prototype`属性。此属性是一个引用类型（指针），它指向函数的原型集对象。我们一般可以通过`prototype`修改函数的原型属性。而**`__proto__`**是一个对象（可以是某个构造器的实例）的属性，它在对象（实例）被创建时跟随被创建，它也是一个引用类型（指针），指向函数（构造器）的`prototype`属性。

两者的关系如下，

```javascript
function Foo() {

}

var foo = new Foo();

console.log(foo.__proto__ === Foo.prototype); // true
```

额外提一点，`__proto__`属性目前在IE浏览器中貌似不能直接访问，不过在Chrome和Firefox中是可以直接访问的。

# `new`的过程

如下代码，

```javascript
function Foo() {

}

var foo = new Foo();
```

我们使用`new`创建了构造器`Foo`的一个实例`foo`。那么这个过程究竟是怎么样的呢？它将分为以下几步，

- 创建一个对象`foo`，并且`foo = {}`
- 将构造器的原型链到`foo`的`__proto__`上，即`foo.__proto__ = Foo.prototype`。这一步至关重要，为*实例能够访问构造器原型方法*及*原型链查找*等操作做好了铺垫。
- 执行构造器`Foo`，并将对象`foo`绑定到其上下文环境中，即`Foo.call(foo)`。其实就是将`Foo`中的`this`指针指向`foo`。

# 示例

## 第一个示例

下面让我们来看个完整的例子来探索一下`prototype`和`__proto`属性的相互关系，

```javascript
function Animal() {
    this.eats = true;
}

var cat = new Animal();
Animal.prototype.jumps = true;

console.log(cat.eats); // true;
console.log(cat.jumps); // true


// change constructor's prototype
Animal.prototype = {
    bark: true
};

var dog = new Animal();

console.log(cat.bark); // undefined
console.log(dog.bark); // true

console.log(cat.__proto === Animal.prototype); // false;
console.log(dog.__proto === Animal.prototype); // true;
```

下面是一张关于上述代码中`Animal`（构造器）及两个实例（`cat`及`dog`）的原型关系图，

![](http://7xkwt1.com1.z0.glb.clouddn.com/区分JS中的__proto__和prototype-001.png)

从图中我们可以看出，

- `cat`和`dog`是`Animal`构造出的实例
- `cat`的`__proto__`属性指向一个对象，此对象是`Animal`改变prototype之**前**的指向。
- `dog`的`__proto__`属性指向一个对象，此对象是`Animal`改变prototype之**后**的指向。和`Animal.prototype`的指向一致。
- 通过`cat`或者`dog`的`__proto__`属性，经过层层查找，最终的都会指向同一个对象，即`Object.prototype`，而`Object.prototype`的`__proto__`指向`null`，此时原型链已经到顶。

## 第二个示例

下面我们再来看个例子来探索一下javascript是如何在其原型链上进行属性查找的，

```javascript
function Person() {
    this.name = 'gejiawen';

    this.secret = function() {
        console.log('I am a secret!');
    };
};

Person.prototype.sayName = function() {
    console.log('My name is ', this.name);
};
Person.prototype.secret = function() {
    console.log('I am a secret on prototype!');
};

var p = new Person();

console.log(p.name); // gejiawen
console.log(p.secret()); // I am a secret!
console.log(p.sayName()); // My name is gejiawen
```

下面是实例对象`p`在Chrome console中的打印，

![](http://7xkwt1.com1.z0.glb.clouddn.com/区分JS中的__proto__和prototype-002.png)

所以，各个打印的解释如下，

- `p.name`，直接在对象`p`中找到了，将其打印出来。
- `p.secret`，在对象`p`中也直接找到了，执行相应方法。
- `p.sayName`，在对象`p`中并未找到相关定义，此时将会开始检索`p.__proto__`中对象，发现了`sayName`的定义，此时就终止检索，执行相应函数。

有两点需要额外提出来，

1. 我们可以看到在构造器和`prototype`中定义了一个同名方法`secret`，从上述代码中可以看出，是不会执行`prototype`中的`secret`函数。因为javascript在示例对象`p`自身的定义找到`secret`时就终止检索了。
2. 针对上面的第三点解释，如果我们在`p.__proto__`仍然未找到`sayName`的相关定义，那么javascript会继续向上检索，继续检查`p.__proto__.__proto__`，如此反复，直至到`Object.prototype`。

## 示例总结

通过上面的示例，我们可以看出，示例对象的`__proto__`在javascript的原型继承模型中扮演着不可或缺的角色。

可以说`__proto__`就是将一个个的对象串成一条完整原型链的粘合器。

个人觉得`prototype`相对于`__proto__`更加像是一个开放的接口，而`__proto__`更倾向是原型链模型的内部实现。我们在平时进行javascript开发时，可以在适当的时候应用一些prototype的知识来提高代码质量。

# 一些额外的问题

## `Object.create`方法

`Object.create`是ES5中引入的方法，

```javascript
Object.create(proto[, propertiesObject])
```

[MDN上](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/create)给出的定义如下，

> The `Object.create()` method creates a new object with the specified prototype object and properties.

意思就是我们可以在创建对象时，可以自定义其原型。（需要注意的是，当给`Object.create`传入的参数不是一个对象或者`null`时，它将会抛出一个错误）

看下面的代码，

```javascript
var animal = {
    eats: true
};

var rabbit = Object.create(animal);

console.log(rabbit.eats); // true
```

此时，`rabbit.__proto__`上将会有`eats`属性，如关系如下图，

![](http://7xkwt1.com1.z0.glb.clouddn.com/区分JS中的__proto__和prototype-003.png)

请注意各个对象的`__proto__`之间的关系。


## `instanceof`背后的逻辑

我们知道`instanceof`可以判断某个实例对象是否是某个构造器的实例。那么`instanceof`判断的依据是什么呢？

```javascript
function A() {

}

var a = new A();

console.log(a instanceof A); // true
```

`instanceof`的过程如下，

1. 对比`a.__proto__`和`A.prototype`。
2. 若第一步的返回为`true`，则表示`a`即为`A`的实例。
3. 若第一步的返回为`false`，此时执行类似这样的操作`a = a.__proto__`，然后重复第一步。

其实说白了，`instanceof`的实质就是比较`__proto__`和`prototype`，我们来看个示例来看下是否能说明这个问题，

```javascript
function Animal() {
   this.name = 'wangcai';
}

function Dog() {

}

Dog.prototype =  new Animal();

var dog = new Dog();

console.log(dog.name); // 'wangcai'
console.log(dog instanceof Animal); // true

// change the prototype of Class
Dog.prototype = {
  name: 'yingcai'
};

console.log(dog instanceof Dog); // false
console.log(dog instanceof Animal); // true
console.log(Animal.prototype.__proto__ === Object.prototype); // true
```

这个例子中，我们中间改变了`Dog`的`prototype`，此时造成的后果就是破坏了原先`dog`对象的`__proto__`与`Dog.prototype`的引用关系，从而`dog instanceof Dog`返回的结果为`false`。


# 参考列表

- [JS的prototype和__proto__](http://www.cnblogs.com/yangjinjin/archive/2013/02/01/2889103.html)
- [JS进阶：`__proto__` 和 `prototype`](http://zhiye.li/2014-02-11-_proto__-prototype-shareslide.html)
- [MDN:Object.create](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/create)




