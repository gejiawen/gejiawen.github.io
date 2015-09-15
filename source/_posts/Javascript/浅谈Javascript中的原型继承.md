postid: "prototype-inherit-in-javascript"
title: 浅谈Javascript中的原型继承
date: 2014-10-16 12:55:27
categories: [Javascript]
tags: [javascript, 翻译]
---

英文原文： [Javascript: An Exploration of Prototypal Inheritance](http://mrjoelkemp.com/2014/01/javascript-an-exploration-of-prototypal-inheritance/)

在Javascript面向对象编程中，原型继承不仅是一个重点也是一个不容易掌握的点。在本文中，我们将对Javascript中的原型继承进行一些探索。

# 基本形式

我们先来看下面一段代码,

```javascript
//构造器函数
function Shape(){
    this.x = 0;
    this.y = 0;
}

//一个shape实例
var s = new Shape();
```

虽然这个例子非常简单，但是有四个**非常重要**的点需要在此阐明：

1. `s`是一个对象，并且默认的它拥有访问`Shape.prototype`（即每个由`Shape`构造函数创建的对象拥有的原型）的权限；简单来说，`Shape.prototype`就是一个**监视**着所有`Shape`实例的对象。你可以将一个对象的原型想象成一个由许多属性（变量/函数）组成的后备集合，当原型在它自己身上找不到东西时就会去原型中查找。
2. 原型可以在所有的`Shape`实例中共享。例如，所有的原型都拥有（直接）访问原型的权限。
3. 当你调用实例中的一个函数时，这个实例会在它自己身上查找这个函数的定义。如果找不到，那么原型将会查找这个函数的定义。
4. 无论被调用的函数的定义在哪里找到（在实例本身中或者它的原型中），`this`的值都是指向用来调用函数的这个实例。因此如果我们调用了一个`s`中的函数，如果这个函数并没有在`s`中直接定义而是在`s`的原型中，`this`值依然指向`s`。

现在我们将上面强调的几点运用到一个例子中。假设我们将一个函数`getPosition()`绑定到`s`上。我们可能会这样做：

```javascript
s.getPosition = function() {
    return [this.x, this.y];
};
```

这样做没有什么错误。你可以直接调用`s.getPosition()`然后你将获得返回的数组。

但是如果我们创建了另一个`Shape`的实例`s2`怎么办呢，它依然能够调用`getPosition()`函数吗？

答案**显然是不能**。

`getPosition`函数直接在实例`s`中被创建。因此，这个函数并不会存在与`s2`中。

当你调用`s2.getPosition()`时，下面的步骤会依次发生(**注意第三步非常重要**)：

1. 实例`s2`会检查`getPosition`的定义；
2. 这个函数不存在于`s2`中；
3. `s2`的原型（和`s`一起共享的后备集合）检查`getPosition`的定义；
4. 这个函数不存在与原型中；
5. 这个函数的定义没有被找到；


一个简单（但并不是最优）的解决方案是将`getPosition`在实例`s2`（以及后面每一个需要`getPosition`的实例）中再定义一次。这是一个很不好的做法因为你在做无意义的复制代码的工作，而且在每个实例中定义一个函数会消耗更多的内存（如果你关心这点的话）。

**我们有更好的办法。**

# 在原型中定义属性

我们完全可以达到所有实例共享`getPosition`函数的目的，不是在每个实例中都定义`getPosition`，而是定义在构造器函数的原型中。我们来看下面的代码：

```javascript
//构造器函数
function Shape(){
    this.x = 0;
    this.y = 0 ;
}

Shape.prototype.getPosition = function(){
    return [this.x, this.y];
};

var s = new Shape(),
    s2 = new Shape();
```

由于原型在所有`Shape`的实例中共享，s和s2都能够访问到`getPosition`函数。

调用`s2.getPosition()`函数会经历下面的步骤：

1. 实例`s2`检查`getPosition`的定义；
2. 函数不存在与`s2`中；
3. 检查原型；
4. `getPosition`的定义存在于原型中；
5. `getPosition`会连同指向`s2`的`this`一起执行；

绑定到原型的属性非常适合于重用。你可以在所有的实例中重用同样的函数。

## 原型中的陷阱

当你把对象或者数组绑定到原型中的时候要非常小心。**所有的实例将会共享这些被绑定的对象/数组的引用**。如果一个实例操纵了对象或数组，那么所有的实例都会受到影响。

```javascript
function Shape() {
    this.x = 0;
    this.y = 0;
}

Shape.prototype.types = ['round', 'flat'];

s = new Shape();
s2 = new Shape();

s.types.push('bumpy');

console.log(s.types); // ['round', 'flat', 'bumpy']
console.log(s2.types); // ['round', 'flat', 'bumpy']
```

当`s.types.push(’bumpy’)`这行代码运行时，实例s将会检查一个叫做`types`的数组。它不存在与实例`s`中，于是原型检查这个数组。这个数组，`types`，存在于原型中，因此我们为他添加一个元素`bumpy`。

结果，由于`s2`也共享原型，它也能通过非直接的方式发现`types`数组发生了变化。

一个真实的例子就是当你使用`Backbone.js`时也会发生类似的事情。当你定义了一个视图/模型/集合，`Backbone`会把你通过`extend`函数(例如：`Backbone.View.extend({})`)传递的属性添加到你定义的实体的原型中。

这意味着如果你在定义实体时添加了一个对象或者数组，所有的实例将会共享这些对象或者数组，很有可能你的一个实例会毁掉另外一个实例。为了避免这样的情况，你经常会看到人们将这些对象/数组包含在一个函数中，每次返回一个对象/数组的实例。

注意：`Backbone`在[model defaults](http://backbonejs.org/#Model-defaults)的部分中谈到了这一点：

> 记住在JavaScript中，对象是以引用的方式被传递的，因此如果你包含了一个对象作为默认值，它将在所有实例中被共享。因此，我们将defaults定义为一个函数。

## 另一种类型的Shape

假设现在我们想要创建一种特定类型的`Shape`，比如说一个圆。如果它能继承`Shape`的所有功能并且还能在它的原型中定义自定义函数那该多好：

```javascript
function Shape() {
    this.x = 0;
    this.y = 0;
}

function Circle() {
    this.radius = 0;
}
```

那么我们怎么形容一个`circle`是一个`shape`呢？有以下几种方法：

### 借用构造函数并且赋值给原型

当我们创建一个圆时，我们想要让实例拥有一个半径（来源于Circle构造函数），以及一个x位置，一个y位置（来源于Shape构造函数）。

我们我们仅仅声明`c = new Circle()`，那么`c`仅仅只有半径。`Shape`构造函数对`x`和`y`进行了初始化。我们想要这个功能。因此我们来借用这个功能。

```javascript
function Circle() {
    this.radius = 0;
    Shape.call(this);
}
```

最后一行代码`Shape.call(this)`调用了`Shape`构造函数并改变了当`Circle`构造函数被调用时指向`this`的值。

好吧，这到底是在说些什么呢？

现在我们来使用上面的构造函数创建一个新的圆然后看看发生了什么：

```javascript
var c = new Circle();
```

这行代码调用了`Circle`构造函数，它首先在`c`上绑定了一个变量`radius`。记住，此时的`this`指向的是`c`。我们接着调用`Shape`构造函数，然后将`Shape`中的`this`值指向当前在`Circle`中的`this`值，也就是`c`。`Shape`构造函数将`x`和`y`绑定到了当前的`this`上，也就是说，`c`现在拥有值为0的`x`和`y`属性。

另外，你在这个例子中放置`Shape.call(this)`的位置并不重要。如果你想在初始化之后**重载x和y**（也就是将圆心放在一个另外的地方），你可以在调用Shape函数之后完成这件事。

问题是现在我们实例化的圆虽然拥有了变量`x`，`y`和`radius`，但是它并不能从`Shape`的原型中获取任何东西。我们需要设置`Circle`构造函数来将`Shape`的原型重置为它的原型 – 以便所有的圆都能获取作为`shape`的福利。

一种方式是我们将`Circle.prototype`的值设置为`Shape.prototype`:

```javascript
function Shape() {
    this.x = 0;
    this.y = 0;
}

Shape.prototype.getPosition = function () {
    return [this.x, this.y];
};

function Circle() {
    this.radius = 0;
    Shape.call(this);
}

Circle.prototype = Shape.prototype; // 注意看这里

var s = new Shape(),
    c = new Circle();
```

这样做运行的很好，但是它并不是最优选择。实例`c`现在拥有访问`getPosition`函数的权限，因为`Circle`构造器函数和`Shape`构造器函数共享了它的原型。

要是我们还想给所有的`Cirecle`定义一个`getArea`函数怎么办？我们将把这个函数绑定到`Circle`构造器函数的原型中以便它可以为所有圆的实例所用。

编写下面的代码：

```javascript
function Shape() {
  this.x = 0;
  this.y = 0;
}

Shape.prototype.getPosition = function () {
  return [this.x, this.y];
};

function Circle() {
    this.radius = 0;
    Shape.call(this);
}

Circle.prototype = Shape.prototype;

Circle.prototype.getArea = function () {
    return Math.PI * this.radius * this.radius;
};

var s = new Shape(),
    c = new Circle();
```

现在的情况是`Circle`和`Shape`共享同一个原型，我们在`Circle.prototype`中添加了一个函数其实也就相当于在`Shape.prototype`中添加了一个函数。

**怎么会这个样子！！**

一个`Shape`的实例并没有`radius`变量，只有`Circle`实例拥有`radius`变量。但是现在，所有的`Shape`实例都可以访问`getArea`函数 – 这将导致一个错误，但是当所有圆调用这个函数时则一切正常。

将所有的原型设置为同一个对象并不能满足我们的需求。


### Circle原型是一个Shape的实例

```javascript
function Shape() {
    this.x = 0;
    this.y = 0;
}

Shape.prototype.getPosition = function () {
    return [this.x, this.y];
};

function Circle() {
    this.radius = 0;
}

Circle.prototype = new Shape();

var c = new Circle();
```

这个方法非常的酷。我们并没有借用构造器函数但是`Circle`拥有了`x`和`y`，同时也拥有了`getPosition`函数。它是怎么实现的呢？

`Circle.prototype`现在是一个`Shape`的实例。这意味着`c`有一个直接的变量`radius`（由Circle构造器函数提供）。然而，在`c`的原型中，有一个`x`和`y`。现在注意，有趣的东西要来了：在`c`的原型的原型中，有一个`getPosition`函数的定义。看起来其实是这样的：

![](http://7xkwt1.com1.z0.glb.clouddn.com/浅谈Javascript中的原型继承-001.png)

因此，如果你试图获取`c.x`，那么它将在`c`的原型中被找到。

这种方法的**缺点**是如果你想要重载`x`和`y`，那么你必须在`Circle`构造器或者`Circle`原型中做这件事。

```javascript
function Shape() {
    this.x = 0;
    this.y = 0;
}

Shape.prototype.getPosition = function () {
    return [this.x, this.y];
};

function Circle() {
    this.radius = 0;
}

Circle.prototype = new Shape();
Circle.prototype.x = 5;
Circle.prototype.y = 10;

var c = new Circle();

console.log(c.getPosition()); // [5, 10]
```

调用`c.getPosition`将会经历下列步骤：

1. 该函数在`c`中没有被找到；
2. 该函数在`c`的原型（`Shape`的实例）中没有被找到；
3. 该函数在`Shape`实例的原型（`c`的原型的原型）中被找到；
4. 该函数连同指向`c`的`this`一起被调用；
5. 在`getPosition`函数的定义中，我们在`this`中寻找`x`；
6. `x`没有直接在`c`中被找到；
7. 我们在`c`的原型（`Shape`实例）中查找`x`；
8. 我们在`c`的原型中找到`x`；
9. 我们在`c`的原型中找到`y`；

除了有一层一层的原型链带来的头痛之外，这个方法还是很不错的。

这个方法还可以使用`Object.create()`来替代。

### 借用构造函数并使用Object.create

```javascript
function Shape() {
    this.x = 0;
    this.y = 0;
}

Shape.prototype.getPosition = function () {
    return [this.x, this.y];
};

function Circle() {
    this.radius = 0;

    Shape.call(this);
    this.x = 5;
    this.y = 10;
}

Circle.prototype = Object.create(Shape.prototype);
Circle.prototype.constructor = Circle; // 这一步是将Circle的原型的构造函数重置为Circle

var c = new Circle();

console.log(c.getPosition()); // [5, 10]
```

这个方法的一大好处就是`x`和`y`直接被绑定到了`c`上 – 这将使查询速度大大提高（如果你的程序关心这件事情）因为你再也不需要向上查询原型链了。

我们来看一看`Object.create`的替代方法([polyfill](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/create#Polyfill))：

```javascript
Object.create = (function() {
    // 中间构造函数
    function F() {}

    return function(o) {
        ...
        // 将中间构造函数的原型设置为我们给它的对象o
        F.prototype = o;
        // 返回一个中间构造函数的实例；
        // 它是一个空对象但是原型是我们给它的对象o
        return new F();
    };
})();
```

上说过程基本上是完成了`Circle.prototype = new Shape()`；只是现在`Circle.prototype`是一个空对象（一个中间构造函数F的实例），而它的原型是`Shape.prototype`。

### 你应该使用哪个方法

非常重要的一点是，如果你在`Shape`构造函数上绑定有对象/数组，那么所有的圆都可以修改这些共享的对象/数组。如果将`Circle.prototype`设置为一个`Shape`的实例时这个方法会有很大的缺陷。

```javascript
function Shape() {
    this.x = 0;
    this.y = 0;
    this.types = ['flat', 'round'];
}

Shape.prototype.getPosition = function () {
    return [this.x, this.y];
};

function Circle() {
    this.radius = 0;
}

Circle.prototype = new Shape();

var c = new Circle(),
    c2 = new Circle();

c.types.push('bumpy');

console.log(c.types);  // ["flat", "round", "bumpy"]
console.log(c2.types); // ["flat", "round", "bumpy"]
```

为了避免这种情况的发生，你可以借用`Shape`的构造函数并且使用`Object.create`以便每一个圆都能拥有它自己的`types`数组。

```javascript
...
function Circle() {
    this.radius = 0;
    Shape.call(this);
}

Circle.prototype = Object.create(Shape.prototype);

var c = new Circle(),
    c2 = new Circle();

c.types.push('bumpy');

console.log(c.types);  // ["flat", "round", "bumpy"]
console.log(c2.types); // ["flat", "round"]
```

## 一个更高级的例子

我们现在在前面讨论的基础上更进一步，创建一个新的`Circle`的类型，`Sphere`。一个椭圆，和圆差不多，只是在计算面积时有不同的公式。

```javascript
function Shape() {
    this.x = 0;
    this.y = 0;
}

Shape.prototype.getPosition = function () {
    return [this.x, this.y];
};

function Circle() {
    this.radius = 0;

    Shape.call(this);
    this.x = 5;
    this.y = 10;
}

Circle.prototype = Object.create(Shape.prototype);

Circle.prototype.getArea = function () {
    return Math.PI * this.radius * this.radius;
};

function Sphere() {

}

// TODO: 在这里设置原型链

Sphere.prototype.getArea = function () {
    return 4 * Math.PI * this.radius * this.radius;
};

var sp = new Sphere();
```

我们应该使用哪种方法来设置原型链？记住，我们并不想要毁掉我们关于圆的`getArea`的定义。我们只是想在椭圆中有另一种方式的实现。

我们不能够借用构造函数并为原型赋值，因为这样做将会改变所有圆的`getArea`的定义。然而，我们可以使用`Object.create`或者将`Sphere`的原型设置为一个`Circle`的实例。我们来看看应该怎么做：

```javascript
...
function Circle() {
    this.radius = 0;

    Shape.call(this);
    this.x = 5;
    this.y = 10;
}

Circle.prototype = Object.create(Shape.prototype);

Circle.prototype.getArea = function () {
    return Math.PI * this.radius * this.radius;
};

function Sphere() {
    Circle.call(this);
}

Sphere.prototype = Object.create(Circle.prototype);

Sphere.prototype.getArea = function () {
    return 4 * Math.PI * this.radius * this.radius;
};

var sp = new Sphere();
```

调用`sp.getArea()`将会经历一下步骤：

1. 在`sp`中查找`getArea`的定义；
2. 在`sp`中没有找到相关定义；
3. 在`Sphere`的原型（一个中间对象，它的原型是`Circle.prototype`）中查找；
4. 在这个中间对象中找到关于`getArea`的定义，由于我们在`Sphere`的原型中重新定义了`getArea`，这里采用新的定义；
5. 连同指向`sp`的`this`调用`getArea`方法；

我们注意到`Circle.prototype`也有一个getArea的定义。然而，由于`Sphere.prototype`已经有了一个`getArea`的定义，我们永远不会使用到`Circle.prototype`中的的`getArea` – 这样我们就成功的**重载**了这个函数（重载意味着在查询链的前面定义了一个名字相同的函数）。


End. All rights reserved `@gejiawen`.


