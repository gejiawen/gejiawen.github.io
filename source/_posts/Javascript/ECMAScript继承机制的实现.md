title: ECMAScript继承实现
date: 2014-09-29 15:42:06
categories: [Javascript]
tags: [javascript]
---

# 实现

[ECMAScript](http://zh.wikipedia.org/wiki/ECMAScript)实现继承机制，可以从要继承的基类入手。所有开发者定义的类都可作为基类。出于安全原因，本地类和宿主类不能作为基类，这样可以防止公用访问编译过的浏览器级的代码，因为这些代码可以被用于恶意攻击。

选定基类后，就可以创建它的子类了。是否使用基类完全由你决定。有时，你可能想创建一个不能直接使用的基类，它只是用于给子类提供通用的函数。在这种情况下，基类被看作抽象类。

尽管ECMAScript并没有像其他语言那样严格地定义抽象类，但有时它的确会创建一些不允许使用的类。通常，我们称这种类为**抽象类**。

创建的子类将继承超类的所有属性和方法，包括构造函数及方法的实现。记住，所有属性和方法都是公用的，因此子类可直接访问这些方法。子类还可添加超类中没有的新属性和方法，也可以覆盖超类的属性和方法。

# 继承的方法

和其他功能一样，ECMAScript实现继承的方式不止一种。这是因为**JavaScript中的继承机制并不是明确规定的**，而是通过模仿实现的。这意味着所有的继承细节并非完全由解释程序处理。作为开发者，你有权决定最适用的继承方式。

## 方法一：对象冒充

构想原始的ECMAScript时，根本没打算设计**对象冒充**（object masquerading）。它是在开发者开始理解函数的工作方式，尤其是如何在函数环境中使用`this`关键字后才发展出来。

其原理如下，

构造函数使用`this`关键字给所有属性和方法赋值（即采用类声明的构造函数方式）。因为构造函数只是一个函数，所以可使`ClassA`构造函数成为`ClassB`的方法，然后调用它。`ClassB`就会收到`ClassA`的构造函数中定义的属性和方法。

例如，用下面的方式定义ClassA和ClassB，

```javascript
function ClassA(sColor) {
    this.color = sColor;
    this.sayColor = function() {
        console.log(this.color);
    };
}

function ClassB() {

}
```

还记得吗？关键字`this`引用的是构造函数当前创建的对象。不过在这个方法中，`this`指向的所属的对象。这个原理是把`ClassA`作为常规函数来建立继承机制，而不是作为构造函数。

如下使用构造函数 ClassB 可以实现继承机制，

```javascript
function ClassB(sColor) {
    this.newMethod = ClassA;
    this.newMethod(sColor);

    delete this.newMethod;
}
```

在这段代码中，为`ClassA`赋予了方法`newMethod`（请记住，函数名只是指向它的指针）。然后调用该方法，传递给它的是`ClassB`构造函数的参数`sColor`。最后一行代码删除了对`ClassA`的引用，这样以后就不能再调用它。

所有新属性和新方法都必须在删除了新方法的代码行后定义。否则，可能会覆盖超类的相关属性和方法，

```javascript
function ClassB(sColor, sName) {
    this.newMethod = ClassA;
    this.newMethod(sColor);

    delete this.newMethod;

    this.name = sName;
    this.sayName = function() {
        console.log(this.name);
    };
}
```

为证明前面的代码有效，可以运行下面的例子，

```javascript
var objA = new ClassA('blue');
var objB = new ClassB('red', 'loda');

objA.sayColor();

objB.sayColor();
objB.sayName();
```

有趣的是，对象冒充可以支持多重继承。也就是说，一个类可以继承多个超类。

例如，如果存在两个类 ClassX 和 ClassY，ClassZ 想继承这两个类，可以使用下面的代，

```javascript
function ClassZ() {
    this.newMethod = ClassX;
    this.newMethod();

    delete this.newMethod;

    this.newMehtod = ClassY;
    this.newMethod();

    delete this.newMethod;
}
```

**这里存在一个弊端，如果存在两个类`ClassX`和`ClassY`具有同名的属性或方法，`ClassY`具有高优先级。**

因为它从后面的类继承。除这点小问题之外，用对象冒充实现多重继承机制轻而易举。

## 方法二：`call()`方法

`call()`方法是与经典的对象冒充方法最相似的方法。它的第一个参数用作`this`的对象。其他参数都直接传递给函数自身。例如，

```javascript
function sayColor(sPrefix, sSuffix) {
    console.log(sPrefix + this.color + sSuffix);
}

var obj = new Object();

obj.color = 'blue';
sayColor.call(obj, 'The color is', 'a very nice color indeed');
```

在这个例子中，函数`sayColor()`在对象外定义，即使它不属于任何对象，也可以引用关键字`this`。对象`obj`的`color`属性等于`blue`。
调用`call()`方法时，第一个参数是`obj`，说明应该赋予`sayColor()`函数中的`this`关键字值是`obj`。
第二个和第三个参数是字符串。它们与`sayColor()`函数中的参数`sPrefix`和`sSuffix`匹配，最后生成的消息`"The color is blue, a very nice color indeed."`将被显示出来。

要与继承机制的对象冒充方法一起使用该方法，只需将前三行的赋值、调用和删除代码替换即可，

```javascript
function ClassB(sColor, sName) {
    //this.newMethod = ClassA;
    //this.newMethod(color);
    //delete this.newMethod;

    ClassA.call(this, sColor);
    this.name = sName;
    this.sayName = function () {
        alert(this.name);
    };
}
```

这里，我们需要让`ClassA`中的关键字`this`等于新创建的`ClassB`对象，因此`this`是第一个参数。第二个参数`sColor`对两个类来说都是唯一的参数。

## 方法三：`apply()`方法

apply() 方法有两个参数，用作 this 的对象和要传递给函数的参数的数组。例如，

```javascript
function sayColor(sPrefix,sSuffix) {
    alert(sPrefix + this.color + sSuffix);
};

var obj = new Object();
obj.color = "blue";

sayColor.apply(obj, new Array("The color is ", "a very nice color indeed."));
```

这个例子与前面的例子相同，只是现在调用的是`apply()`方法。调用`apply()`方法时，第一个参数仍是`obj`，说明应该赋予`sayColor()`函数中的`this`关键字值是`obj`。
第二个参数是由两个字符串构成的数组，与`sayColor()`函数中的参数`sPrefix`和`sSuffix`匹配，最后生成的消息仍是`"The color is blue, a very nice color indeed."`，将被显示出来。

该方法也用于替换前三行的赋值、调用和删除新方法的代码，

```javascript
function ClassB(sColor, sName) {
    //this.newMethod = ClassA;
    //this.newMethod(color);
    //delete this.newMethod;

    ClassA.apply(this, new Array(sColor));
    this.name = sName;
    this.sayName = function () {
        alert(this.name);
    };
}
```

同样的，第一个参数仍是`this`，第二个参数是只有一个值`color`的数组。可以把`ClassB`的整个`arguments`对象作为第二个参数传递给`apply()`方法，

```javascript
function ClassB(sColor, sName) {
    //this.newMethod = ClassA;
    //this.newMethod(color);
    //delete this.newMethod;

    ClassA.apply(this, arguments);
    this.name = sName;
    this.sayName = function () {
        alert(this.name);
    };
}
```

当然，只有超类中的参数顺序与子类中的参数顺序完全一致时才可以传递参数对象。如果不是，就必须创建一个单独的数组，按照正确的顺序放置参数。此外，还可使用`call()`方法。


## 方法四：原型链（prototype chaining）

继承这种形式在ECMAScript中原本是用于原型链的。`prototype`对象是个模板，要实例化的对象都以这个模板为基础。总而言之，`prototype`对象的任何属性和方法都被传递给那个类的所有实例。原型链利用这种功能来实现继承机制。

如果用原型方式重定义前面例子中的类，它们将变为下列形式，

```javascript
function ClassA() {

}

ClassA.prototype.color = "blue";
ClassA.prototype.sayColor = function () {
    alert(this.color);
};

function ClassB() {

}

ClassB.prototype = new ClassA();
```

原型方式的神奇之处在于上述的最后一行代码。
这里，把`ClassB`的`prototype`属性设置成`ClassA`的实例。这很有意思，因为想要`ClassA`的所有属性和方法，但又不想逐个将它们`ClassB`的`prototype`属性。还有比把`ClassA`的实例赋予`prototype`属性更好的方法吗？

**注意：调用`ClassA`的构造函数，没有给它传递参数。这在原型链中是标准做法。要确保构造函数没有任何参数。**

与对象冒充相似，子类的所有属性和方法都必须出现在`prototype`属性被赋值后，因为在它之前赋值的所有方法都会被删除。
为什么？因为`prototype`属性被替换成了新对象，添加了新方法的原始对象将被销毁。
所以，为`ClassB`类添加`name`属性和`sayName()`方法的代码如下，

```javascript
function ClassB() {

}

ClassB.prototype = new ClassA();
ClassB.prototype.name = "";
ClassB.prototype.sayName = function () {
    alert(this.name);
};
```

可通过运行下面的例子测试这段代码，

```javascript
var objA = new ClassA();
var objB = new ClassB();

objA.color = "blue";
objB.color = "red";
objB.name = "John";

objA.sayColor();
objB.sayColor();
objB.sayName();
```

此外，在原型链中，`instanceof`运算符的运行方式也很独特。对`ClassB`的所有实例，`instanceof`为`ClassA`和`ClassB`都返回`true`。例如，

```javascript
var objB = new ClassB();

alert(objB instanceof ClassA);	//输出 "true"
alert(objB instanceof ClassB);	//输出 "true"
```

在ECMAScript的弱类型世界中，这是极其有用的工具，不过使用对象冒充时不能使用它。

原型链的弊端是不支持多重继承。记住，原型链会用另一类型的对象重写类的`prototype`属性。

## 方式五：混合方式

这种继承方式使用构造函数定义类，并非使用任何原型。对象冒充的主要问题是必须使用构造函数方式，这不是最好的选择。不过如果使用原型链，就无法使用带参数的构造函数了。开发者如何选择呢？答案很简单，两者都用。

在前面的内容中，我们曾经讲解过创建类的最好方式是用构造函数定义属性，用原型定义方法。这种方式同样适用于继承机制，用对象冒充继承构造函数的属性，用原型链继承`prototype`对象的方法。用这两种方式重写前面的例子，代码如下，

```javascript
function ClassA(sColor) {
    this.color = sColor;
}

ClassA.prototype.sayColor = function () {
    alert(this.color);
};

function ClassB(sColor, sName) {
    ClassA.call(this, sColor);
    this.name = sName;
}

ClassB.prototype = new ClassA();
ClassB.prototype.sayName = function () {
    alert(this.name);
};
```

在此例子中，继承机制由两行突出显示的蓝色代码实现。在第一行突出显示的代码中，在`ClassB`构造函数中，用对象冒充继承`ClassA`类的`sColor`属性。在第二行突出显示的代码中，用原型链继承`ClassA`类的方法。由于这种混合方式使用了原型链，所以`instanceof`运算符仍能正确运行。

下面的例子测试了这段代码，

```javascript
var objA = new ClassA("blue");
var objB = new ClassB("red", "John");

objA.sayColor();//输出 "blue"
objB.sayColor();//输出 "red"
objB.sayName();	//输出 "John"
```

## 方式六：ES6中的继承实现

参考[ES6的继承实现](https://github.com/gejiawen/mytest/blob/master/class-inherit/class-inherit.js)。其核心就是**`B.prototype.__proto__ = A.prototype;`**，可以直接将父类的`prototype`赋值给子类`prototype`对象的`__proto__`属性达到对象继承的目的。


End. All rights reserved `@gejiawen`.
