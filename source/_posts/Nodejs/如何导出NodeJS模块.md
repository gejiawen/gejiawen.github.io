title: 如何导出NodeJS模块
date: 2014-10-16 15:55:39
categories: [Nodejs]
tags: [nodejs, 翻译, nodejs-基础入门]

---

英文原文： [Export This: Interface Design Patterns for Node.js Modules](http://bites.goodeggs.com/posts/export-this/)


当你在Node中require一个模块时，你从返回的结果中得到了什么？当你编写一个Node模块时，在设计模块的接口时你有哪些选择？

今天我们将讨论七种Node.js模块接口设计模式，在实际工作中，它们经常会被混合起来使用：

1. 导出一个命名空间
2. 导出一个函数
3. 导出一个高阶函数
4. 导出一个构造函数
5. 导出一个单体
6. 扩展一个全局对象
7. 运用一个猴子补丁(Monkey Patch)

## require,exports和module.exports

首先我们来回顾一下基础。

在Node中，`require`一个文件实际上实在`require`这个文件定义的模块。所有的模块都拥有一个对隐式`module`对象的引用，当你调用`require`时实际上返回的是`module.exports`属性。对于`module.exports`的引用同样也能写成`exports`。

在每一个模块的第一行都隐式的包含了一行下面的代码：

```javascript
var exports = module.exports = {};
```

**注意：**如果你想要导出一个函数，你需要将这个函数赋值给`module.exports`。将一个函数赋值给`exoports`将会为`exports`引用重新赋值，但是`module.exports`依然会指向原始的空对象。

因此我们可以像这样来定义一个`function.js`模块来导出一个对象：

```javascript
module.exports = function(){
    return {name: 'Jane'};
}
```

然后在另一个文件中require这个模块：

```javascript
var fund = require('./function');
```

`require`的一个重要行为就是它缓存了`module.exports`的值并且在未来再次调用`require`时返回同样的值。它依据被`require`文件的绝对路径来进行缓存。因此如果你想要你的模块返回不同的值，你应该导出一个在再次调用时能返回不同值的函数。

为了证明这点，我们在Node REPL中进行一些操作：

```javascript
$ node
> f1 = require('/Users/alon/Projects/export_this/function');
[Function]
> f2 = require('./function'); //同样的位置
> f1 === f2
true
> f1() === f2()
false
```

在上面的例子中，你可以看到`require`返回了同一个函数实例但是由函数调用返回的对象是完全不同的。

更详细的信息你可以参看Node模块系统的文档。

现在我们开始正式进入接口设计模式。


## 导出一个命名空间

一个简单而常用的模式是导出一个拥有若干属性的对象，这些属性主要是函数但是不限于函数。这种方式允许代码通过`require`一个模块在一个命名空间下获取一组相关联的功能。

当你`require`了一个导出命名空间的模块，你通常会把整个命名空间赋值给一个变量来使用它的成员，或者将它的成员直接赋值给本地变量：

```javascript
var fs = require('fs'),
    readFile = fs.readFile,
    ReadStream = fs.ReadStream;

readFile('./file.txt', function(err, data) {
    console.log("readFile contents: '%s'", data);
});

new ReadStream('./file.txt').on('data', function(data) {
    console.log("ReadStream contents: '%s'", data);
});
```

下面是fs核心模块中的一行代码：

```javascript
var fs = exports;
```

它首先将本地变量fs赋值为隐式导出对象`exports`然后将函数引用赋值为fs的属性。因为fs指向`exports`并且`exports`是当你调用require(’fs’)时返回的对象，因此任何赋值给fs的东西在你通过`require`获取的对象中都可用。

```javascript
fs.readFile = function(path,options,callback_){
    //...
}
```

任何东西都是一个公平的游戏。它接下来会导出一个构造函数：

```javascript
fs.ReadStream = ReadStream;

function ReadStream(path,options){
    //...
}

ReadStream.prototype.open = function(){
    //...
}
```

当导出一个命名空间时，你可以将属性赋值于给`exports`或者fs模块，或者将一个新对象复制给`module.exports`。

```javascript
module.exports = {
    version: '1.0',

    doSomething: function() {
        //...
    }
};
```

导出一个命名空间的普遍用法是导出其他模块的根对象以便一次`require`就能够获取若干个模块。在我之前的项目Good Eggs中，我们将每个分开的子模块都出了一个模型构造函数并且接着编写了一个能导出所有模型的index文件。这允许我们可以在一个`models`命名空间下获取所有的`model`。

```javascript
var models = require('./models'),
    User = models.User,
    Product = models.Product;
```

对于CoffeeScript用户，析构赋值（[restructuring assignment](http://coffeescript.org/#destructuring)）使得这个工作更加轻松了：

```javascript
{User, Product} = require './models'
```

`index.js`文件看起来是这样的：

```javascript
exports.User = require('./user');
exports.Person = require('./person');
```

事实上，我们使用一个小巧的库来require所有的子文件并且将它们使用驼峰命名法导出以便`index.js`文件实际上能够读取下面内容：

```javasript
module.exports = require('../lib/require_siblings')(__filename);
```

## 导出一个函数

另一个模式是导出一个函数作为一个模块的接口。一个普遍的用法是导出一个在调用时能返回一个兑现高的工厂函数。在使用`Express.js`时我们这样编写代码：

```javascript
var express = require('express');
var app = express();

app.get('/hello', function (req, res) {
    res.send "Hi there! We're using Express v" + express.version;
});
```

由Express导出的这个函数被用来创建一个新的Express应用。在你自己使用这种模式时，你的工厂函数可能需要接收一些参数来配置或者初始化返回的对象。

为了导出一个函数，你需要将你的函数赋值给`module.exports`。

```javascript
exports = module.exports = createApplication;
...
function createApplication () {
  ...
}
```

上面的例子将`createApplication`函数赋值给了`module.exports`然后赋值给隐式的`exports`变量。现在`exports`就是模块导出的函数。

Express中同样将这个导出的函数作为命令空间来使用。

```javascript
exports.version = '3.1.1';
```

要注意的一点是没有什么阻止我们将导出的函数作为命令空间使用，它能够暴露出对于其他函数、构造函数或者对象的引用。

当导出一个函数时，最佳实践是位这个函数命名以便它能在栈追踪中出现。注意到下面两个例子的的栈追踪的不同之处：

```javascript
// bomb1.js
module.exports = function () {
    throw new Error('boom');
};

// bomb2.js
module.exports = function bomb() {
    throw new Error('boom');
};

$ node
> bomb = require('./bomb1');
[Function]
> bomb()
Error: boom
    at module.exports (/Users/alon/Projects/export_this/bomb1.js:2:9)
    at repl:1:2
    ...
> bomb = require('./bomb2');
[Function: bomb]
> bomb()
Error: boom
    at bomb (/Users/alon/Projects/export_this/bomb2.js:2:9)
    at repl:1:2
    ...
```

在导出一个函数的情形中有许多值得特别说明的点。


## 导出一个高阶函数

一个高阶函数，或者函子(functor)，是一个接收一个或多个函数作为输入或者输出的函数。我们将讨论后面一种情形 – 即一个**返回函数的函数**。

当你想要从你的模块返回一个函数但是需要获取控制函数行为的输入时，导出一个高阶函数是一个非常有用的模式。

Connect中间件提供了许多对于Express和其他web框架的插件功能。一个中间件就是一个接收三个参数 – `(req`,`res`,`next)` – 的函数。这样的用法在connect中间件中是为了导出一个在调用时返回一个中间件函数的函数。这允许导出的函数接收能够被用于配置中间件以及在中间件的闭包作用域中可用的变量，当它在处理一个请求时。

例如，connect中的`query`中间件在Express中被哟关于解析查询字符串变量。

```javascript
var connect = require('connect'),
    query = require('connect/lib/middleware/query');

var app = connect();
app.use(query({maxKeys: 100}));
```

query模块的源代码如下所示：

```javascript
var qs = require('qs'),
    parse = require('../utils').parseUrl;

module.exports = function query(options) {
    return function query(req, res, next) {
        if (!req.query) {
            req.query = ~req.url.indexOf('?') ? qs.parse(parse(req).query, options) : {};
        }

        next();
    };
};
```

对于每一个通过query中间件的请求，在整个闭包作用域中都可用的`options`参数将单独传递给Node的核心模块qs模块。

这个设计模式是你在工作中非常常用且非常灵活的一个模式。

## 导出一个构造函数

我们在JavaScript以构造函数的方式定义类并且使用`new`关键字创建类的实例。

```javascript
function Person(name) {
    this.name = name;
}

Person.prototype.greet = function() {
    return "Hi, I'm Jane.";
};

var person = new Person('Jane');
console.log(person.greet()); // prints: Hi, I'm Jane
```

这种设计模式实现了**一个文件一个类**并且使得你的项目组织结构更加清晰，使得其他的开发者能够轻松发现类的实现方式。

```javascript
var Person = require('./person');

var person = new Person('Jane');
```

实现的方式如下所示:

```javascript
function Person(name) {
    this.name = name;
}

Person.prototype.greet = function() {
    return "Hi, I'm " + this.name;
};

module.exports = Person;
```

## 导出一个单体

当你想要你的模块的所有用户来分享一个类的实例的状态和行为时你需要导出一个单体。

Mongoose是一个对象-文档映射库，它被用来创建永久保存在MongoDB中的富结构域对象。


```javascript
var mongoose = require('mongoose');
mongoose.connect('mongodb://localhost/test');

var Cat = mongoose.model('Cat', { name: String });

var kitty = new Cat({ name: 'Zildjian' });
kitty.save(function (err) {
    if (err) {
        // ...
    }
    console.log('meow');
});
```

当我们require Mongoose时返回的mongoose对象是什么？在内部，mongoose模块是这样的：

```javascript
function Mongoose() {
    //...
}

module.exports = exports = new Mongoose();
```

由于require缓存了所有赋值给`module.exports`的值，所有对于`require('mongoose')`的调用那个将会返回同一个实例以确保它在我们的应用中是一个单体。Mongoose使用面向对象设计模式来压缩及解耦功能，保持状态并且支持可读性与可理解行，但是通过创建并导出Mongoose类的一个实例来创建一个面向用户的简单接口。

如果用户需要，Mongoose也会将这个单体实例作为命名空间来使用以确保其他的构造函数也可以使用，其中包括Mongoose构造函数本身。你可能需要使用Mongoose构造器函数来创建连接到其他MongoDB数据库的实例。

在内部，Mongoose是这样做的：

```javascript
Mongoose.prototype.Mongoose = Mongoose;
```

因此你可以这样做：

```javascript
var mongoose = require('mongoose'),
    Mongoose = mongoose.Mongoose;

var myMongoose = new Mongoose();
myMongoose.connect('mongodb://localhost/test');
```

## 扩展一个全局对象

一个被require的模块能做的不仅仅是导出一个值。他可能够修改全局对象或者require其他模块时返回的对象。它可以定义一个新的全局对象。它可以只扩展一个对象或者在扩展一个全局对象的基础上导出一些有用的东西。

当你需要在你的对象中扩展或者改变全局对象的行为时你需要使用这个模式。虽然饱含争议并且应该谨慎使用（尤其是在开源项目中），该模式确是一个必不可少的模式。

`Should.js`是一个在单元测试中使用的断言库。

```javascript
require('should');

var user = {
    name: 'Jane'
};

user.name.should.equal('Jane');
```

Should.js通过在对象中扩展了一个不可枚举的属性`should`来为单元测试中编写断言提供一个清晰的语法。在内部，should.js是这么做的：

```javascript
var should = function(obj) {
    return new Assertion(util.isWrapperType(obj) ? obj.valueOf(): obj);
};

//...

exports = module.exports = should;

//...

Object.defineProperty(Object.prototype, 'should', {
    set: function(){},
    get: function(){
        return should(this);
    },
    configurable: true
});
```

注意到`Should.js`导出了一个`should`函数，它的主要用途是为`Object`添加`should`属性。


## 运用一个猴子补丁

这里说的**猴子补丁**指的是*在运行过程中对类或者模块进行动态的修改，目的是为了给第三方代码添加一个补丁*。

当一个存在的模块没有提供你需要的功能时你可以实现一个模块作为它的补丁。这个模式是前面一个模式的变形。它并不是像上一个模式一样对全局对象进行修改，而是依靠Node模块系统的缓存行为对一个模块的同一个实例添加补丁以便当该模块被其他代码require时仍然能返回修改过的对象。

默认情形下Mongoose会将MongoDB的集合以小写和复数来命名。对于一个叫做`CreditCardAccountEntry`的集合最终存储在MongoDB中的名字叫做`creditcardaccountentries`。但是我想要它的名字为`credit_card_account_entries`并且我想要全局使用这种行为。

下面是一个针对mongoose.model的补丁模块：

```javascript
var Mongoose = require('mongoose').Mongoose;
var _ = require('underscore');

var model = Mongoose.prototype.model;
var modelWithUnderScoreCollectionName = function(name, schema, collection, skipInit) {
    collection = collection || _(name).chain().underscore().pluralize().value();
    model.call(this, name, schema, collection, skipInit);
};
Mongoose.prototype.model = modelWithUnderScoreCollectionName;
```

当这个模块第一次被require时，它require了mongoose，重定义了Mongoose.prototype.model并且将它代理返回原生的model。现在所有Mongoose的实例都将拥有新的欣行为。注意到它并没有修改exports因此通过require返回的值还是默认的空exports对象。

另外，如果你选择对已有模块运用一个猴子补丁，最好使用上面例子中的链式技巧。你在猴子补丁中添加你的行为然后这些行为将会代理回到原生的实现方式。虽然这种方式并不是很简单，但是它是对于第三方代码最好的添加补丁的方式，它允许你利用未来库的升级并且将你的补丁和其他补丁的冲突降低到最小。

## 总结

Node模块系统对于封装功能以及创建清晰的接口提供了一种非常简单的机制。希望上面提到的几种设计模式对于你有所帮助。


End. All rights reserved `@gejiawen`.


