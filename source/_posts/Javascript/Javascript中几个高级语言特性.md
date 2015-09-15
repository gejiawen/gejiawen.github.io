postid: "some-javascript-advanced-feature"
title: Javascript中几个高级语言特性
date: 2014-09-19 17:12:58
categories: [Javascript]
tags: [javascript]
---

*感谢Node.js开发指南，参考了它的附录部分内容。*

# 作用域

Javascript中的作用域是通过函数来确定的，这一点与`C`、`Java`等静态语言有一些不一样的地方。

## 最简单的例子

```javascript
if (true) {
    var a =  'Value';
}
console.log(a); // Value
```
上面的代码片段将会输出Value。（在浏览器环境中）

## 更加common的例子

再来一个更加common的例子，

```javascript
var a1 = 'Valve';
var foo1 = function() {
    console.log(a1);
};
foo1(); // Value
var foo2 = function() {
    var a1 = 'DOTA2';
    console.log(a1);
}
foo2(); // DOTA2
```

显然，`foo1`的结果是Value，`foo2`的结果是DOTA2，这应该很容易理解。

## 有点迷惑的例子

接下来这个例子将会让人感到迷惑，

```javascript
var a1 = 'mercurial';
var foo = function() {
    console.log(a1);
    var a1 = 'git';
}
foo(); // undefined
```

此时，结果将会是`undefined`。

<!-- more -->

因为在函数foo内部的a1将会覆盖函数外部的变量a1，js搜索作用域是按照从内到外的，而且当执行到console.log时，函数作用域内部的a1还尚未被初始化，所以会输出undefined。

其实这里还涉及到一个*变量悬置*的概念，即在Javascript的函数中，无论在何处声明或者初始化的变量都等效于函数的起始位置声明，在实际位置赋值。如下，

```javascript
var foo = function() {
    // do something
    var a = 'ok';
    console.log(a);
    // do something
}
```

上面这段代码等效于，

```javascript
var foo = function() {
    var a; // 注意看这里！
    // do something
    a = 'ok';
    console.log(a);
    // do something
}
```

最后还有一点需要说明的就是，**未定义变量**和**定义但未被初始化的变量**，虽然他们的值输出都是undefined，但是在js内部的实现上还是有区别的。未定义的变量存在于js的局部语义表上，但是未被分配内存，而定义却未初始化的变量时实际分配了内存的。

## 嵌套作用域

接下来这个例子将会演示函数作用域的嵌套，

```javascript
var foo = function() {
    var a1 = 'foo';
    (function() {
        var a1 = 'foo1';
        (function() {
            console.log(a1);
        })();
    })();
};
foo(); // foo1
```

输出结果是foo1。这里我在最内层的`console.log`中打印a1，此时，因为最内层的作用域中没有a1的相关定义，所以会往上层作用域搜索，得到a1='foo1'。这里实际上有一个嵌套的作用域关系。

## 静态作用域

这里还有一点需要注意，就是函数作用的嵌套关系是在定义时就会确定的，而非调用的时候。也即js的作用域是**静态作用域**，好像又叫**词法作用域**，因为**在代码做语法分析时就确定下来了**。看下面的这个例子，

```javascript
var a1 = 'global';
var foo1 = function() {
    console.log(a1);
};
foo1(); // global
var foo2 = function() {
    var a1 = 'locale';
    foo1();
};
foo2(); // global
```

示例的输出结果都将会是`global`。`foo1()`的执行结果为`global`不需要太多的解释，很容易明白。

因为`foo2`在执行时，调用`foo1`，`foo1`方法会从他自己的作用域开始搜索变量`a1`，最终在其父级作用域中找到`a1`，即`a1 = 'global'`。由此可以看出，`foo2`内部的`foo1`在执行时并没有去拿`foo2`作用域中的变量`a1`。

以说**作用域的嵌套关系并不是在执行时确定的，而是在定义时就确定好了的！**

## 全局作用域

最后提一下全局作用域。通过字面的意思就能知道，全局作用域中的变量也好，属性也好，在任何函数中都能直接访问。

其中有一点需要注意，在任何地方没有通过var关键字声明的变量都是定义在全局变量中。其实，在模块化编程中，应该尽量避免使用全局变量，声明变量时，无论如何都应该避免**不**使用var关键字。


# 闭包

闭包是函数式编程语言的一大语言特性。w3c上关于闭包的严格定义如下：**由函数(环境)及其封闭的自由变量组成的集合体**。这句话比较晦涩难懂，反正刚开始我是没看懂。下面通过一些例子来说明。

## 闭包解释

```javascript
var closure = function() {
    var count = 0;
    return function() {
        count ++;
        return count;
    };
};
var counter = closure();
console.log(counter()); // 1
console.log(counter()); // 2
console.log(counter()); // 3
```

最后的结果是1,2,3。

这个demo中，`closure`是一个函数(其实他相当于一个类的构造函数)，并且返回一个函数(这个*被返回的函数*加上*其定义环境*通俗上被称为**闭包**)。
在返回的函数中，引用了外部的`count`变量。在`var counter = closure();`这句代码之后，`counter`实际上就是一个函数，这样每次在`counter()`时，先将count自增然后打印出来。
这里`counter`的函数内部并没有关于`count`的定义，所以在执行时会往上层作用域搜索，而他的上层作用域是`closure`函数，而不是`counter()`执行时所在的上层作用域。

为什么它的上层作用域是`closure`函数呢？因为，

- 第一，这是在定义的时候就已经确定好的函数作用域嵌套关系，
- 更重要的是第二点，闭包的返回不但有函数而且还包含定义函数的上下文环境。这里上下文环境就是`closure`函数的内部作用域，所以能够拿到`closure`函数中的`count`变量。

从这里可以看出，闭包会造成对原作用域和其上层作用域的**持续引用**。在这里，`count`变量持续被引用，其所占用的内存就不会被释放掉。

在看下面的这个例子，

```javascript
var closure = function() {
    var count = 0;
    return function() {
        count ++;
        return count;
    };
};
var counter1 = closure();
var counter2 = closure();

console.log(counter1()); // 1
console.log(counter1()); // 2
console.log(counter2()); // 1
console.log(counter2()); // 2
console.log(counter1()); // 3
```

从结果可以看出，生成的闭包实例是各自独立的，他们内部引用的`count`变量分别属于各自不同的运行环境。
我们可以这样理解，在闭包生成时，将原上下文环境做了一份拷贝副本，这样不同的闭包实例就有自己独立的运行环境了。

## 闭包的应用场景

闭包目前来说有两大用处，
- 第一是嵌套的回调函数
- 第二是隐藏对象的部分细节

```javascript
$('#id0').animate({
    left: '+50px'
}, 1000, function() {
    $('#id1').animate({
        left: '+50px'
    }, 1000, function() {
        $('#id2').animate({
            left: '+50px'
        }, 1000, function() {
            alert('done');
        });
    });
});
```

Javascript的对象没有私有成员的概念。一般的编码规范中会要求类似`_privateProp`的形式来定义私有属性。但是这是一个非正式的约定，而且`_privateProp`仍然能够被访问到。

我们可以通过闭包来实现私有成员，如下，

```javascript
var student = function(yourName, yourAge) {
    var name, age;

    name = yourName || '';
    age = yourAge || 0;

    return {
        getName: function() {
            return name;
        },
        getAge: function() {
            return age;
        },
        setName: function(yourName) {
            name = yourName;
        },
        setAge: function(yourAge) {
            age = yourAge;
        }
    };
}

var mamamiya = student('mamamiya', 23);
mamamiya.getName();
mamamiya.getAge();
```

这里我封装了一个`student`类，并设置了两个属性`name`，`age`。这两个属性除了通过`student`对象的访问器方法访问之外，绝无其他的方法能够访问到。这里就实现了对部分属性的隐藏。

# 对象

Javascript的对象是基于原型的，和其他的一些面向对象语言有一些区别。

## 创建和访问

我们可以通过如下的这种形式来创建一个js对象。

```javascript
var foo = {
    'a': 'baz',
    'b': 'foz',
    'c': function() {
        return 'hello js';
    }
};
```

我们还可以通过构造函数来创建对象。

```javascript
function user(name, uri) {
    this.name = name;
    this.uri = uri;
    this.show = function() {
        console.log(this.name);
    }
};

var mamamiya = new user('mamamiya', 'http://gejiawen.github.io');
mamamiya.show();
```

Javascript中上下文对象就是`this`，他表示被调用函数所处的环境。他的作用就是在一个函数内部引用调用它自己。

在Javascript中，任何函数都是被某个对象调用。


## `apply`和`call`

在Javascript中`apply`和`call`是两个神奇的方法，他们的作用是以不同的上下文环境来调用函数。通俗点就是说，**一个对象可以调用另一个对象的方法**。

看下面的例子，

```javascript
var user = {
    name: 'mamamiya',
    show: function(words) {
        console.log(this.name + ' says ' + words);
    }
};
var foo = {
    name: 'baz'
};

user.show.call(foo, 'hello'); // baz says hello
```

这段代码的结果是`baz says hello`。这里通过`call`方法改变了`user.show`方法的上下文环境，`user.show`方法在执行时，内部的`this`指向的是`foo`对象。

## `bind`方法

可以使用`bind`方法永久的改变函数的上下文。`bind`将会返回一个函数引用。

看下面的这个例子，

```javascript
var user = {
    name: 'mamamiya',
    func: function() {
        console.log(this.name);
    }
};
var foo = {
    name: 'baz'
};

foo.func = user.func;
foo.func(); //baz

foo.func1 = user.func.bind(user);
foo.func1(); //mamamiya

func = user.func.bind(foo);
func(); //baz
    
func2 = func;
func2(); //baz
```

其实，`bind`还可以在绑定上下文时附带一些参数。

不过有时候，`bind`会有一些让人迷惑的地方，看下面这个例子，

```javascript
var user = {
    name: 'mamamiya',
    func: function() {
        console.log(this.name);
    }
};
var foo = {
    name: 'baz'
};
func  = user.func.bind(foo);
func(); //baz
func2 = func.bind(user);
func2(); //baz
```

这里为什么`func2`函数的输出结果仍然是`baz`呢？

也就是说，我企图将`func`的上下文环境还原到`user`上为什么没有起作用？

我们这样来看，

```javascript
func = user.func.bind(foo) ≈ function() {
    return user.func.call(foo);
};

func2 = func.bind(user) = function() {
    return func.call(user);
};
```

ok，现在可以看出来，`func2`中实际上是以`user`为`this`指针调用了`func`，但是在`func`中并没有使用`this`。

## prototype

通过构造函数和原型都能生成对象，但是两者之间有一些区别。看下面的这个列子，

```javascript
function Class() {
    var a = 'hello';
    this.prop1 = 'git';
    this.func1 = function() {
        a = '';
    };
}

Class.prototype.prop2 = 'Mercurial';

Class.prototype.func2 = function() {
    console.log(this.prop2);
};

var class1 = new Class();
var class2 = new Class();

console.log(class1.func1 === class2.func1); //false
console.log(class1.func2 === class2.func2); //true
```

所以说，挂在`prototype`上的属性，会被不同的实例会共享。通过构造函数创建出来的属性，每一个实例都有一份独立的副本。

## 原型及基于原型的面向对象

那么，**什么叫原型链？**

JavaScript中有两个特殊的对象：`Object`与`Function`，它们都是构造函数，用于生成对象。
`Object.prototype`是所有对象的祖先，`Function.prototype`是所有函数的原型，包括构造函数。

我把JavaScript中的对象分为三类，

- 一类是用户创建的对象，即一般意义上用`new`语句显式构造的对象
- 一类是构造函数对象，即普通的构造函数，通过`new`调用生成普通对象的函数
- 一类是原型对象，即构造函数`prototype`属性指向的对象

这三类对象中每一种都有一个`__proto__`属性，**它指向该对象的原型**。任何对象沿着它开始遍历都可以追溯到`Object.prototype`。

构造函数对象有`prototype`属性，指向一个原型对象，通过该构造函数创建对象时，被创建对象的`__proto__`属性将会指向构造函数的`prototype`属性。原型对象有`constructor`属性，指向它对应的构造函数。

看下面的这个例子，帮助理解，

```javascript
function foo() { }

Object.prototype.name = 'My Object';
foo.prototype.name = 'baz';

var obj = new Object();
var foo = new foo();
console.log(obj.name); // My Object
console.log(foo.name); // baz
console.log(foo.__proto__.name); // baz
console.log(foo.__proto__.__proto__.name); // My Object
console.log(foo.__proto__.constructor.prototype.name); // baz
```

在Javascript中，继承是依靠一套叫做原型链的机制实现的。
说的通俗一点就是，在继承的时候，将父类的实例对象直接赋值给子类的`prototype`对象，这样子类就拥有了父类的全部属性。子类还可以在自己的prototype对象上增加自己的特殊属性。

看下面的例子，

```javascript
function ClassA() { }

ClassA.prototype.color = "blue";
ClassA.prototype.sayColor = function () {
    alert(this.color);
};

function ClassB() { }

ClassB.prototype = new ClassA();
```

## 对象的复制

Javascript中所有的对象类型的变量都是指向对象的引用。所以在赋值和传递的实际上都是对象的引用。

在Javascript中，对象的复制分为**浅拷贝**和**深拷贝**。

下面的示例是浅拷贝，

```javascript
Object.prototype.makeCopy = funciton() {
    var newObj = {};
    for (var i in this) {
        newObj[i] = this[i];
    }
    return newObj;
};

var obj = {
    name: 'mamamiya',
    likes: ['js']
};

var newObj = obj.makeCopy();
obj.likes.push('python');

console.log(obj.likes); // ['js', 'python']
console.log(newObj.likes); // ['js', 'python']
```

从上面的代码可以看出，浅拷贝只是复制了一些基本属性，但是对象类型的属性是被共享的。`obj.likes`和`newObj.likes`都指向同一个数组。

想要做深拷贝，并不是一件容易的事情，因为除了基本数据类型，还有多种不同的对象，对象内部还有复杂的结构，因此需要用递归的方式来实现。

看下面的例子，

```javascript
Object.prototype.makeDeepCopy = function() {
    var newObj = {};
    for (var i in this) {
        if (typeof(this[i]) === 'object' || typeof(this[i]) === 'function') {
            newObj[i] = this[i].makeDeepCopy();
        } else {
            newObj[i] = this[i];
        }
    }
    return newObj;
};

Array.prototype.makeDeepCopy = function() {
    var newArray = [];
    for (var i = 0; i < this.length; i++) {
        if (typeof(this[i]) === 'object' || typeof(this[i]) === 'function') {
            newArray[i] = this[i].makeDeepCopy();
        } else {
            newArray[i] = this[i];
        }
    }
    return newArray;
};

Function.prototype.makeDeepCopy = function() {
    var self = this;
    var newFunc = function() {
        return self.apply(this, arguments);
    }
    for (var i in this) {
        newFunc[i] = this[i];
    }
    return newFunc;
};

var obj = {
    name: 'mamamiya',
    likes: ['js'],
    show: function() {
        console.log(this.name);
    }
};

var newObj = obj.makeDeepCopy();
newObj.likes.push('python');
console.log(obj.likes); // ['js']
console.log(newObj.likes); // ['js', 'python']
console.log(newObj.show == obj.show); // false
```

上面的示例代码中很好的实现了对象，函数，数组在做深拷贝的逻辑。在一般情况下都是比较好用的。但是有一种情况下，这种方法却无能为力。如下：

```javascript
var obj1 = {
    ref: null
};
var obj2 = {
    ref: obj1
};
obj1.ref = obj2;
```

上面这段代码块的逻辑很简单，就是两个相互引用的对象。

当我们试图使用深拷贝来复制`obj1`和`obj2`中的任何一个时，问题就出现了。因为深拷贝的做法是遇到对象就进行递归复制，那么结果只能无限循环下去。

对于这种情况，简单的递归已经无法解决，必须设计一套图论算法，分析对象之间的依赖关系，建立一个拓扑结构图，然后分别依次复制每个顶点，并重新构建它们之间的依赖关系。这已经超出了这里的讨论范围，而且在实际的工程操作中 几乎不会遇到这种需求，所以我们就不继续讨论了。

