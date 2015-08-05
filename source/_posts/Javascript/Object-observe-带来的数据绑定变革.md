title: Object.observe()带来的数据绑定变革
date: 2014-10-30 15:48:29
categories: [Javascript]
tags: [es6, javascript, 翻译, 转载]
---

- 英文原文： [Data-binding Revolutions with Object.observe()](http://www.html5rocks.com/en/tutorials/es7/observe/) （需翻墙）
- [参考](http://div.io/topic/600)


# 引言

一场变革即将到来。Javascript中的一项新特性将会颠覆之前你对于**数据绑定**的所有认识。它也将改变你所使用的MVC库观察模型中发生的修改以及更新的实现方式。你会看到，那些所有在意属性观察的应用性能将会得到巨大的提升。

我们很高兴的看到，`Object.observe()`已经正式加入到了Chrome 36 beta版本中。

`Object.observe()`是未来ECMAScript标准之一，它是一个可以**异步观察Javascript中对象变化的方法**，而无需使用一个其他的JS库。它允许一个观察者接收一个**按照时间排序的变化记录序列**，这个序列描述的是一列被观察的对象所发生的变化。

```javascript
// Let's say we have a model with data
var model = {};

// Which we then observe
Object.observe(model, function(changes){

    // This asynchronous callback runs
    changes.forEach(function(change) {

        // Letting us know what changed
        console.log(change.type, change.name, change.oldValue);
    });

});
```

当被观察的对象发生任何变化时，回调函数将会汇报这些变化：

![](http://7xkwt1.com1.z0.glb.clouddn.com/Object-observe-带来的数据绑定变革-001.png)

通过使用`Object.observe()`，你可以[不需要使用任何框架](http://bitworking.org/news/2014/05/zero_framework_manifesto)就能实现双向数据绑定。

但这并不意味着你就不应该使用一个框架。对于一些有着复杂业务逻辑的项目，经过精心设计的框架的重要性不言而喻，你应该继续使用它们。这些框架减轻了开发新手的工作量，而且只需要编写很少的代码就能够维护和实现一些模式并达到我们想要的目的。如果你不需要一个框架，你可以使用一个体积更小，针对性更强的库，比如[Polymer](http://polymer-project.org/)（它已经开始使用`Object.observe()`了）。

即便你已经重度依赖于一个`MV*`框架，`Object.observe()`仍然能为你的应用带来一些性能各方面的提升，它能够更快更简单的实现一些功能并维持同样的API。例如，在Angular以前的一个Benchmark测试中，对于一个Model中发生的变化，脏值检查对每次更新会花费40ms，而`Object.observe()`只会花费1-2ms（相当于20-40倍的性能提升）。

不需要冗长代码来实现的数据双向绑定还意味着你不需要通过轮询来发现变化，这将带来更长的电池使用时间！

如果你已经对`Object.observe()`有了一些了解，可以直接跳过简介这一节，或者接着阅读了解更多关于它能够解决的问题。

# 我们要需要观察什么？

当我们在讨论数据观察时，我们通常指的是对一些特定类型的数据变化保持关注：

- 原始JavaScript对象中的变化
- 当属性被添加、改变、或者删除时的变化
- 当数组中的元素被添加或者删除时的变化
- 对象的原型发生的变化

# 数据绑定的重要性

当你开始关心**模型-视图**的控制分离时，数据绑定就会变成一件重要的事。HTML是一个非常好的声明机制，但是它完全是静态的。理想状态下，你想要在数据和DOM之间声明它们的关系，以便让DOM保持更新。这会让你节省很多由于写一些重复代码而浪费的时间。

当你拥有一个复杂的用户界面，你需要理清楚许多数据属性和许多视图中元素的关系时，数据绑定是非常有用的。这在我们今天需要创建的单页应用中非常常见。

通过在浏览器中原生的观察数据，我们给予了JavaScript框架（或者你编写的一些功能库）一种方式来实现对模型数据的变化进行观察而不需要依赖于我们今天正在使用的一些hack方法。

# 现在使用的数据绑定方案是什么样子的

## 脏值检查

你以前曾经在那里看到过数据绑定？如果你在你的web应用中使用过一个现代MV*框架(例如Angular，Knockout)，那么你或许已经使用过数据绑定将数据绑定到你的DOM上了。为了复习一下，下面是一个电话列表应用的例子，在其中我们会将一个phones数组中的值（在JavaScript中定义）绑定到一个列表项目中以便于我们的数据和UI保持同步。

```html
<html ng-app>
    <head>
        ...
        <script src="angular.js"></script>
        <script src="controller.js"></script>
    </head>
    <body ng-controller="PhoneListCtrl">
        <ul>
            <li ng-repeat="phone in phones">
                {{phone.name}}
                <p>{{phone.snippet}}</p>
            </li>
        </ul>
    </body>
</html>
```

js的controller如下：

```javascript
var phonecatApp = angular.module('phonecatApp', []);

phonecatApp.controller('PhoneListCtrl', function($scope) {
    $scope.phones = [
        {
            'name': 'Nexus S',
            'snippet': 'Fast just got faster with Nexus S.'
        }, {
            'name': 'Motorola XOOM with Wi-Fi',
            'snippet': 'The Next, Next Generation tablet.'
        }, {
            'name': 'MOTOROLA XOOM',
            'snippet': 'The Next, Next Generation tablet.'
        }
    ];
});
```

demo的地址在[这里](http://angular.github.io/angular-phonecat/step-2/app/)。

在任何时候，只要是底层的model数据发生了变化，我们在DOM中的列表也会跟着更新。Angular是怎么做到这一点的呢？在Angular的背后，有一个叫做**脏值检查**的东西。

![](http://7xkwt1.com1.z0.glb.clouddn.com/Object-observe-带来的数据绑定变革-002.png)

脏值检查的基本原理就是只要任何时候数据发生了变化，这个库都会通过一个**`digest`**或者**`change cycle`**去检查变化是否发生了。在Angular中，一个`digest`循环意味着所有被监视的表达式都会被循环一遍以便查看其中是否有变化发生。它[知道](http://stackoverflow.com/questions/9682092/databinding-in-angularjs/9693933#9693933)一个模型之前的值，因此当变化发生时一个change事件将会被触发。对于开发者来说，这带来的一大好处就是你可以使用原生的JavaScript对象数据，它易于使用及整合。下面的图片展示的是一个非常糟糕的算法，它的开销非常大。

![](http://7xkwt1.com1.z0.glb.clouddn.com/Object-observe-带来的数据绑定变革-003.png)

这个操作的开销和被监视的对象的数量是成正比的。我们可能需要做很多的脏治检查。同时我也需要一种方式去触发脏值检查，当某些数据可能发生改变时。有很多的框架使用了一些非常聪明的方法来解决这个问题，但是它们是否足够好目前还尚无定论。

web生态系统应该拥有更多的能力去创新和进化它自己的声明机制，例如：

- 有约束的模型系统
- 自动的保存系统（例如：将变化保存在IndexedDB或者localStorage中）
- 容器对象（Ember，Backbone）

[容器对象](http://www.slideshare.net/mixonic/containers-di)是一个框架创建的对象，它能够在其中保存一些数据。它们拥有一些存取器去获取数据并且能够在你设置或者获取对象时捕获到这些行为并在内部进行广播。这是一种非常好的方式。它的性能很好，从算法上来说也不错。下面是一个使用Ember容器对象的一个简单例子：

```javascript
// Container objects
MyApp.president = Ember.Object.create({
  name: "Barack Obama"
});

MyApp.country = Ember.Object.create({
  // ending a property with "Binding" tells Ember to
  // create a binding to the presidentName property
  presidentNameBinding: "MyApp.president.name"
});

// Later, after Ember has resolved bindings
MyApp.country.get("presidentName");
// "Barack Obama"

// Data from the server needs to be converted
// Composes poorly with existing code
```

在上面的例子中，发现什么地方发生了变化的开销和发生改变的东西有着直接联系。现在你存在的另一个问题是你需要使用不同种类的对象。总的来说你需要将从服务器获取的数据进行转换以便它们是能够被观察到的。

目前的JS代码并不能很好的整合生成数据，因为这些代码一般会假设它们操作的是原生JavaScript对象，而不是一些特定的对象类似类型。

# 介绍Object.observe()

我们真正想要的可能是两个世界中最好的东西 -- 一种支持对原生数据对象（普通JavaScript对象）进行观察的方法，同时不需要每次都对所有东西进行脏值检查。它需要有良好的算法表现。它还需要能够很好的整合到各个平台中。这些都是`Object.observe()`能够带给我们的东西。

它允许我们对一个对象或者变异属性进行观察，并且在变化发生时得到及时通知。但是我们在这里不想看什么理论，让我们来看看代码！

![](http://7xkwt1.com1.z0.glb.clouddn.com/Object-observe-带来的数据绑定变革-004.png)


## Object.observe()和Object.unobserve()

让我们假设我们现在有一个简单的JavaScript对象，它代表一个模型：

```javascript
// A model can be a simple vanilla object
var todoModel = {
    label: 'Default',
    completed: false
};
```

我们可以制定一个比回调函数，用来处理对象上的变化：

```javascript
function observer(changes){
    changes.forEach(function(change, i){
        console.log('what property changed? ' + change.name);
        console.log('how did it change? ' + change.type);
        console.log('whats the current value? ' + change.object[change.name]);
        console.log(change); // all changes
    });
}
```

**注意：**当观察者回调函数被调用时，被观察的对象可能已经发生了多次改变，因此对于每一次变化，新的值（即每次变化以后的值）和当前值（最终的值）并不一定是相同的。

我们可以使用`Object.observe()`来观察这些变化，只要将对象作为第一个参数，而将回调函数作为第二个参数：

```javascript
Object.observe(todoModel, observer);
```

我们现在对我们的Todos的模型对象做一些改变：

```javascript
todoModel.label = 'Buy some more milk';
```

看看控制台，我们现在得到了一些有用的信息！我们知道什么属性发生了变化，它是怎样变化的以及新的值是什么。

![](http://7xkwt1.com1.z0.glb.clouddn.com/Object-observe-带来的数据绑定变革-005.png)

再见，脏值检查！你的墓碑应该被刻上Comic Sans字体。我们再来改变其他的属性。这次改变的是`completeBy`:

```javascript
todoModel.completeBy = '01/01/2014';
```

正如我们所见的，我们又再一次得到了关于变化的报告：

![](http://7xkwt1.com1.z0.glb.clouddn.com/Object-observe-带来的数据绑定变革-006.png)

非常好。要是我们现在决定从对象中删除`completed`属性会怎么样：

```javascript
delete todoModel.completed;
```

![](http://7xkwt1.com1.z0.glb.clouddn.com/Object-observe-带来的数据绑定变革-007.png)

正如我们所见的，返回的变化报告包含了关于删除的信息。正如我们所期待的，新的值现在是`undefined`。那么，我们现在知道了你可以知道属性什么时候被添加。什么时候被删除。基本上来说，你可以知道一个对象上的属性集（'new','deleted','recongigured'）以及它的原型(proto)的变化。

在任何观察系统中，总是存在一个方法来停止观察。在这里，我们有`Object.unobserve()`方法，它的用法和`Object.observe()`一样但是可以像下面一样被调用：

```javascript
Object.unobserve(todoModel, observer);
```

正如下面所示，在使用该方法之后，任何的变化都不再作为一个变化列表记录返回。

![](http://7xkwt1.com1.z0.glb.clouddn.com/Object-observe-带来的数据绑定变革-008.png)

## 指定感兴趣的变化

现在我们已经了解到了我们如何去获取一个被观察对象的变化列表。但是如果我们仅仅只对一个对象中的某些属性感兴趣该怎么办？人人都需要一个垃圾邮件过滤器。`Observer`可以通过一个列表指定一些我们想要看到的变化。我们需要通过`Object.observe()`的**第三个参数**来指定：

```javascript
Object.observe(obj, callback, opt_acceptList)
```

现在我们来看一个如何使用的例子：

```javascript
// Like earlier, a model can be a simple vanilla object

var todoModel = {
    label: 'Default',
    completed: false

};


// We then specify a callback for whenever mutations
// are made to the object
function observer(changes){
    changes.forEach(function(change, i){
        console.log(change);
    });
};

// Which we then observe, specifying an array of change
// types we’re interested in

Object.observe(todoModel, observer, ['delete']);

// without this third option, the change types provided
// default to intrinsic types

todoModel.label = 'Buy some milk';

// note that no changes were reported
```

如果我们删除了这个标签，注意到这个类型的变化将会被报告：

```javascript
delete todoModel.label;
```

如果你不指定一个列表，它默认将会报告**固有的**对象变化类型 ("add", "update", "delete", "reconfigure", "preventExtensions" (丢与那些不可扩展的对象是不可观察的))。

# 通知

`Object.observe()`也带有一些通知。它们并不像是你在手机上看到了通知，而是更加有有用。通知和变异观察者比较类似。它们发生在微任务的结尾。在浏览器的上下文，它几乎总是位于当前事件处理器的结尾。

这个时间点非常的重要因为基本上来说此时一个工作单元已经结束了，现在观察者已经开始它们的共走了。这是一个非常好的回合处理模型。

使用一个通知器的工作流程如下所示:

![](http://7xkwt1.com1.z0.glb.clouddn.com/Object-observe-带来的数据绑定变革-009.png)

现在我们通过一个例子来如何通过自定义一个通知器来处理一个对象的属性被设置或者被获取的情况。注意看代码中的注释：

```javascript
// Define a simple model
var model = {
    a: {}
};

// And a separate variable we'll be using for our model's getter in just a moment
// 定义一个单独的变量，我们即将使用它来作为我们的模型中的getter
var _b = 2;

// Define a new property 'b' under 'a' with a custom getter and setter
// 在'a'下面定义一个新的属性'b'，并自定义一个getter和setter

Object.defineProperty(model.a, 'b', {
    get: function () {
        return _b;
    },
    set: function (b) {

        // Whenever 'b' is set on the model
        // notify the world about a specific type
        // of change being made. This gives you a huge
        // amount of control over notifications
        // 当'b'在模型中被设置时，注意一个特定类型的变化将会发生
        // 这将给你许多关于通知的控制器
        Object.getNotifier(this).notify({
            type: 'update',
            name: 'b',
            oldValue: _b
        });

        // Let's also log out the value anytime it gets
        // set for kicks
        // 在值发生变化时将会输出信息
        console.log('set', b);

        _b = b;
    }
});

// Set up our observer
// 设置我们的观察者
function observer(changes) {
    changes.forEach(function (change, i) {
        console.log(change);
    })
}

// Begin observing model.a for changes
Object.observe(model.a, observer);
```

![](http://7xkwt1.com1.z0.glb.clouddn.com/Object-observe-带来的数据绑定变革-010.png)

现在当数据属性发生变化时('update')我们将会得到报告。以及任何对象的实现也将会被报告(`notifier.notifyChange()`)。

多年的web平台开发经验告诉我们整合方法是你应该最先尝试的事情，因为它最容易去实现。但是它存在的问题是以它会创造一个从根本上来看就很未下的处理模型。如果你正在编写代码并且更新了一个对象的属性，你实际上并不想陷入这样一种困境：更新模型中的属性会最终导致任意一段代码去做任意一件事情。当你的函数正好运行到一半时，假设失效并不是什么理想的状况。

如果你是一个观察者，你并不想当某人正在做某事的时候被调用。你并不像在不连续的状态下被调用。因为这最终往往会导致更多的错误检查。你应该试着去容忍更多的情形，并且基本上来说它是一个很难去合作的模型。异步是一件更难处理的事情但是最终它会产生更好的模型。

上述问题的解决办法是变化合成记录(synthetic change records)。

# 变化合成记录

基本上来说，如果你想要存取器或者计算属性的话，你应该复杂在这些值发生改变时发出通知。这会导致一些额外的工作，但是它是这种机制第一类的特征，并且这些通知会连同来自余下的底层数据对象的通知一起被发布出来。

![](http://7xkwt1.com1.z0.glb.clouddn.com/Object-observe-带来的数据绑定变革-011.png)

观察存取器或者计算属性的问题可以通过使用`notifier.notify`来解决 -- 它也是`Object.observe()`的另外一部分。大多数的观察系统想要某些形式的观察导出值。有很多方法可以实现它。`Object.observe()`并没有用*正确的*方式进行判断。计算属性应该是存取器，当内部的（私有的）状态发生改变时它应该发出通知。

再一次声明，在web中应该有一些库来帮助我们进行通知并且帮助我们更好的实现计算属性（以及减少模板的使用）。

我们在这里会假设一个例子，这个例子中有一个`circle`类。在这里，我们有一个`citcle`，它有一个`radius`属性。在这里的情形中，`radius`是一个存取器，并且当它的值发生变化时它实际上会去通知自己值已经发生变化了。这些通知将会连同其他变化被传递到这个对象或者其他对象。本质上来说，如果你正在实现一个对象，你一定会想要拥有整合或者计算属性的对象，或者你想要想出一个策略如何让它运行。一旦你做了这件事，它将会适应你的整个系统。

看看下面的代码在开发者工具中是如何运行的：

```javascript
function Circle(r) {
  var radius = r;

  var notifier = Object.getNotifier(this);
  function notifyAreaAndRadius(radius) {
    notifier.notify({
      type: 'update',
      name: 'radius',
      oldValue: radius
    })
    notifier.notify({
      type: 'update',
      name: 'area',
      oldValue: Math.pow(radius * Math.PI, 2)
    });
  }

  Object.defineProperty(this, 'radius', {
    get: function() {
      return radius;
    },
    set: function(r) {
      if (radius === r)
        return;
      notifyAreaAndRadius(radius);
      radius = r;
    }
  });

  Object.defineProperty(this, 'area', {
    get: function() {
      return Math.pow(radius, 2) * Math.PI;
    },
    set: function(a) {
      r = Math.sqrt(a/Math.PI);
      notifyAreaAndRadius(radius);
      radius = r;
    }
  });
}

function observer(changes){
  changes.forEach(function(change, i){
    console.log(change);
  })
}
```

![](http://7xkwt1.com1.z0.glb.clouddn.com/Object-observe-带来的数据绑定变革-012.png)


# 存取器属性

在这里我们对于存取器属性有一个简短的提示。在前面我们提到了对于数据属性来说只有值得变化是能够被观察到的。而存取器属性和计算属性则无法被观察到。这是因为JavaScript中的存取器并没有真正的值的变化。一个存取器仅仅是一个函数集合。

如果你为一个存取器属性赋值,你仅仅只是调用了这个函数，并且在它看来值并没有发生变化。它仅仅只是让一些代码运行起来。

这里的问题在于我们在上面的例子中将存取器属性赋值为5.我们应该能够知道这里究竟发生了什么。这实际上是一个未解决的问题。这个例子说明了原因。对任何系统来说知道这究竟意味着什么是不可能的，因为在这里可以运行任意代码。每当存取器属性被访问时，它的值都会发生改变，因此询问它什么时候会发生变化并没有多大的意义。

# 使用一个回调函数观察多个对象

`Object.observe()`上的另一个模式是使用单个回调观察者。这允许我们使用同一个回调函数堆多个不同的对象进行观察。这个回调函数在“微任务”的结尾将会把所有的变化都传递给它所观察的对象。

![](http://7xkwt1.com1.z0.glb.clouddn.com/Object-observe-带来的数据绑定变革-013.png)

# 大规模的变化

也许你正在编写一个非常大的应用，并且经常需要处理大规模的变化。此时我们希望用一种更加紧凑的方式来描述影响很多属性的语义变化。

`Object.observe()`使用两个特定的函数来解决这个问题：`notifier.performChange()`以及`notifier.notify()`，我们在上面已经介绍过这两个函数了。

![](http://7xkwt1.com1.z0.glb.clouddn.com/Object-observe-带来的数据绑定变革-014.png)

我们可以从下面的例子中看到我们如何来描述大规模变化，在这个例子中定义了一个叫做`Thingy`的对象，其中包含几个数计算功能（multiply, increment, incrementAndMultiply）。只要其中一个功能被使用，它就会告诉系统一些包含特定变化的事情发生了。

例如： `notifier.performChange(‘foo’, performFooChangeFn)`

```javascript
function Thingy(a, b, c) {
  this.a = a;
  this.b = b;
}

Thingy.MULTIPLY = 'multiply';
Thingy.INCREMENT = 'increment';
Thingy.INCREMENT_AND_MULTIPLY = 'incrementAndMultiply';


Thingy.prototype = {
  increment: function(amount) {
    var notifier = Object.getNotifier(this);

    // Tell the system that a collection of work comprises
    // a given changeType. e.g
    // notifier.performChange('foo', performFooChangeFn);
    // notifier.notify('foo', 'fooChangeRecord');
    notifier.performChange(Thingy.INCREMENT, function() {
      this.a += amount;
      this.b += amount;
    }, this);

    notifier.notify({
      object: this,
      type: Thingy.INCREMENT,
      incremented: amount
    });
  },

  multiply: function(amount) {
    var notifier = Object.getNotifier(this);

    notifier.performChange(Thingy.MULTIPLY, function() {
      this.a *= amount;
      this.b *= amount;
    }, this);

    notifier.notify({
      object: this,
      type: Thingy.MULTIPLY,
      multiplied: amount
    });
  },

  incrementAndMultiply: function(incAmount, multAmount) {
    var notifier = Object.getNotifier(this);

    notifier.performChange(Thingy.INCREMENT_AND_MULTIPLY, function() {
      this.increment(incAmount);
      this.multiply(multAmount);
    }, this);

    notifier.notify({
      object: this,
      type: Thingy.INCREMENT_AND_MULTIPLY,
      incremented: incAmount,
      multiplied: multAmount
    });
  }
}
```

我们可以为我们的对象定义两个观察者： 一个用来捕获所有的变化，另一个将只会汇报我们定义的特定类型的变化 (`Thingy.INCREMENT`, `Thingy.MULTIPLY`, `Thingy.INCREMENTANDMULTIPLY`)。

```javascript
var observer, observer2 = {
    records: undefined,
    callbackCount: 0,
    reset: function() {
      this.records = undefined;
      this.callbackCount = 0;
    },
};

observer.callback = function(r) {
    console.log(r);
    observer.records = r;
    observer.callbackCount++;
};

observer2.callback = function(r){
	console.log('Observer 2', r);
}


Thingy.observe = function(thingy, callback) {
  // Object.observe(obj, callback, optAcceptList)
  Object.observe(thingy, callback, [Thingy.INCREMENT,
                                    Thingy.MULTIPLY,
                                    Thingy.INCREMENT_AND_MULTIPLY,
                                    'update']);
}

Thingy.unobserve = function(thingy, callback) {
  Object.unobserve(thingy);
}
```

我们现在可以开始玩弄一下代码了。我们先定义一个新的`Thingy`：

```javascript
var thingy = new Thingy(2,4);
```

对它进行观察并进行一些变化。有趣的事情发生了！

```javascript
// Observe thingy
Object.observe(thingy, observer.callback);
Thingy.observe(thingy, observer2.callback);

// Play with the methods thingy exposes
thingy.increment(3);               // { a: 5, b: 7 }
thingy.b++;                        // { a: 5, b: 8 }
thingy.multiply(2);                // { a: 10, b: 16 }
thingy.a++;                        // { a: 11, b: 16 }
thingy.incrementAndMultiply(2, 2); // { a: 26, b: 36 }
```

![](http://7xkwt1.com1.z0.glb.clouddn.com/Object-observe-带来的数据绑定变革-015.png)

位于这个`perform function`中的一切东西都可以被看作是*大型变化*进行的工作。接受*大型变化*的观察者仅仅只会接受*大型变化”*记录。那些不会接受底层变化的观察者都来源于`perform function`所做的事。

# 观察数组

我们已经讨论了如何观察一个对象，但是应该如何观察数组呢？

`Array.observe()`是一个针对自身大型变化的方法 -- 例如 -- `splice`，`unshift`或者任何能够隐式影响数组长度的东西。在内部它使用了`notifier.performChange`(“splice”,...)。

下面是一个我们如何观察一个模型数组的例子，当底层数据发生一些变化时，我们将能够得到一个变化的列表。

```javascript
var model = ['Buy some milk', 'Learn to code', 'Wear some plaid'];
var count = 0;

Array.observe(model, function(changeRecords) {
  count++;
  console.log('Array observe', changeRecords, count);
});

model[0] = 'Teach Paul Lewis to code';
model[1] = 'Channel your inner Paul Irish';
```

![](http://7xkwt1.com1.z0.glb.clouddn.com/Object-observe-带来的数据绑定变革-016.png)


# 性能

考虑`Object.observe()`性能的方式是将它想成读缓存。基本上来说，在以下几种情形中，一个缓存是最佳选择（按照重要性排序）：

1. 读的频率决定着写的频率
2. 你可以创造一个缓存，它可以在读数据期间将涉及到写数据的操作进行算法上的优化
3. 写数据减慢的时间常数是可以接受的

**`Object.observe()`是为上述第一种情形设计的。**

脏值检查需要保留一个你所要观察数据的副本。这意味着在脏值检查中你需要一个额外的结构内存开销。脏值检查，一个作为权宜之计的解决方案，同时根本上也是一个脆弱的抽象，它可能会导致应用中一些不必要的复杂性。

脏值检查在任何数据可能发生变化的时候都必须要运行。这很明显并不是一个非常鲁棒的方法，并且任何实现脏值检查的途径都是有缺陷的（例如，在轮询中进行检查可能会造成视觉上的假象以及涉及到代码的紊乱情况）。脏值检查也需要注册一个全局的观察者，这很可能会造成内存泄漏，而Object.observe()会避免这一点。

我们现在来看一些数据。

下面的[基准测试](https://github.com/Polymer/observe-js/tree/master/benchmark)允许我们比较**脏值检查**和`Object.observe()`。图中比较的数据是*Observed-Object-Set-Size*和*Number-Of-Mutations*。

总的结果表明：**脏值检查的性能和被观察的对象成正比，而``Object.observe()的性能和我们所做的改变成正比。**

**Dirty-checking**

![](http://7xkwt1.com1.z0.glb.clouddn.com/Object-observe-带来的数据绑定变革-017.png)

**Chrome with Object.observe() switched on**

![](http://7xkwt1.com1.z0.glb.clouddn.com/Object-observe-带来的数据绑定变革-018.png)


# 为Object.observe()提供垫片

`Object.observe()`现在已经可以在Chrome 36 beta中使用，但是如果我们想要在其他浏览器中使用它该怎么办？Polymer中的[Observe-JS](https://github.com/Polymer/observe-js)是一个针对于那些没有原生实现`Object.observe()`浏览器的一个垫片，但是它不仅仅是作为垫片，同时也包含了许多有用的语法糖。它提供了一种整合的视角，它能够将所有变化总结起来并且提交一份关于变化的报告。它的好处主要体现在两点：

1. 你可以**观察路径**。这意味着你可以说，我想要从一个给定的对象中观察`foo.bar.baz`，只要这个路径的值发生了改变，你会得到通知。如果路径是错误的，将会返回`undefined`。

下面是一个例子：

```javascript
var obj = { foo: { bar: 'baz' } };

var observer = new PathObserver(obj, 'foo.bar');
observer.open(function(newValue, oldValue) {
  // respond to obj.foo.bar having changed value.
});
```

2. 它**能够告诉你数组的拼接**。数组拼接基本上来说是你为了将旧版本数组转换为新版本数组是需要进行了最基本的拼接操作。这是一种转换的类型或者是这个数组的不同视图。它是你想要将数组从旧状态变为新状态时需要进行的最基本的工作。

下面是一个例子

```javascript
var arr = [0, 1, 2, 4];

var observer = new ArrayObserver(arr);
observer.open(function(splices) {
  // respond to changes to the elements of arr.
  splices.forEach(function(splice) {
    splice.index; // index position that the change occurred.
    splice.removed; // an array of values representing the sequence of elements which were removed
    splice.addedCount; // the number of elements which were inserted.
  });
});
```

# 框架和Object.observe()

正如上面所提到的，使用`Object.observe()`能够给予框架和库中关于数据绑定的性能巨大的提升。

来自Ember的Yehuda Katz和Erik Bryn已经确定将会在Ember最近的修改版本中添加对`Object.observe()`的支持。来自Angular的Misko Hervy写了一份关于Angular 2.0的设计文档，其中的内容关于改善变化探测（change detection）。在将来，当`Object.observe()`在Chrome稳定版中出现时，Angular会使用`Object.observe()`来实现变化探测的功能，在此之前它们会选择使用[Watchtower.js](https://github.com/angular/watchtower.js/) -- Angular自己的变化探测的实现方式。实在是太令人激动了。

# 总结

`Object.observe()`是一个添加到web平台上非常强大的特性，你现在就可以开始使用它。

我们希望这项特征能够及时的登陆到更多的浏览器中，它能够允许JavaScript框架从本地对象观察的能力中获得更多性能上的提升。Chrome 36 beta及其以上的版本都能使用这项特性，在未来Opera发布的版本中这项特性也会得到支持。

现在就和JavaScript框架作者谈谈`Object.observe()`如何能够提高他们框架中数据绑定的性能。未来还有更多让人激动的时刻。


# 参考链接

- [Object.observe() on the Harmony wiki](http://wiki.ecmascript.org/doku.php?id=harmony:observe)
- [Databinding with Object.observe() by Rick Waldron](http://bocoup.com/weblog/javascript-object-observe/)
- [Everything you wanted to know about Object.observe() - JSConf](http://addyosmani.com/blog/the-future-of-data-binding-is-object-observe/)
- [Why Object.observe() is the best ES7 feature](http://georgestefanis.com/blog/2014/03/25/object-observe-ES7.html)




End. All rights reserved `@gejiawen`.


