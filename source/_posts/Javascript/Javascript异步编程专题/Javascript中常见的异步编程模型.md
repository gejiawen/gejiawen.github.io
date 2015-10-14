postid: 'some-javascript-async-pattern'
title: Javascript中常见的异步编程模型
categories: [Javascript]
tags: [javascript, Javascript异步编程专题]
date: 2015-10-12 15:26:30

---


Javascript异步编程专题，目前包含以下几篇文章，

- [浅谈Javascript中的异步](http://gejiawen.github.io/2015/10/12/think-about-async-in-javascript/)
- [Javascript中常见的异步编程模型](http://gejiawen.github.io/2015/10/12/some-javascript-async-pattern/)
- ES6中的异步编程模型


本篇文章是本专题的第二篇文章。

正文开始。

在Javascript异步编程专题的前一篇文章[浅谈Javascript中的异步](http://gejiawen.github.io/2015/10/12/think-about-async-in-javascript/)中，我简明的阐述了“Javascript中的异步原理”、“Javascript如何在单线程上实现异步调用”以及“Javascript中的定时器”等相关问题。

本篇文章我将会谈一谈Javascript中常用的几种异步编程模型。

在前端的代码编写中，异步的场景随处可见。比如鼠标点击、键盘回车、网络请求等这些与浏览器紧密联系的操作，比如一些延迟交互特效等等。

在这些场景中，你必须要使用所谓的“异步模式”，否则将会严重程序的可行性和用户体验。我们列举这些场景中常用的几种异步编程模型，

- 回调函数
- 事件监听
- 观察者模式（消息订阅/发布）
- promise模式

下面我们将针对每一种编程模型加以说明。

# 回调函数

**回调函数**可以说是Javascript异步编程最基本的方法。我们试想有这样一个场景，我们需要在页面上展示一个持续3秒钟的loading视觉样式，然后在页面上显示我们真正想显示的内容。示例代码如下，

```javascript
// more code

function loading(callback) {
    // 持续3秒的loading展示
    setTimeout(function () {
        callback();
    }, 3000);
}

function show() {
    // 展示真实数据给用户
}

loading(show);

// more code
```

代码中的`loading(show)`就是将函数`show()`作为函数`loading()`的参数。在`loading()`完成3秒的loading之后，再去执行回调函数（示例使用了`setTimeout`来模拟）。通过这种方法，`show()`就变成了异步调用，它的执行时机被推迟到`loading()`即将完成之前。

回调函数往往就是调用用户提供的函数，该函数往往是以参数的形式提供的。回调函数并不一定是异步执行的。回调函数的特点就是使用简单、容易理解。缺点就是逻辑间存在一定耦合。最恶心的地方在于会造成所谓的[callback hell](http://callbackhell.com/)。比如下面这样的一个例子，

```javascript
A(function () {
    B(function () {
        C(function() {
            D(function() {
                // ...
            })
        })
    })
})
```

例子中A、B、C、D四个任务存在依赖关系，通过函数回调的方式，写出来的代码就会变成上面的这个样子。维护性和可读性都非常糟糕。

除了回调嵌套的问题之外，还可能会带来另一个问题，就是流程控制不方便。比如我们要发送3个请求，当3个请求都返回时，我们再执行相关逻辑，那么代码可能就是，

```javascript
var count = 0
for (var i = 0; i < 3; i++) {
    request('source_' + i, function () {
        count++;
        if (count === 3) {
            // do my logic
        }
    });
}
```

上面的示例代码中，我通过`request`对三个url发送了请求，但是我不知道这三个请求的返回情况。无奈之下我添加了一个计数器`count`，在每个请求的回调中都进行计数器判断，当计数器为3时即表示三个请求都已经成功返回了，此时再去执行相关任务。显而易见，这种情况下的流程控制就显得比较丑陋。

最后，有时候我们为了程序的健壮性，可能会需要一个`try...catch`语法。比如，

```javascript
// demo1
try {
    setTimeout(function () {
        throw new Error('error occured');
    })
} catch(e) {
    console.log(e);
}

// demo2
setTimeout(function () {
    try {
        // your logic
    } catch(e) {
        
    }
});
```

上面的示例代码中，如果我们像demo1那样将`try...catch`加在异步逻辑的外面，即使异步调用发生了异常我们也是捕获不到的，因为`try...catch`不能捕获未来的异常。无奈，我们只能像demo2那样将`try...catch`语句块放在具体的异步逻辑内。这样一旦异步调用多起来，那么就会多出来很多`try...catch`。这样肯定是不好的。

# 事件监听

**事件监听**在UI编程中随处可见。比如我给一个按钮绑定一个点击事件，给一个输入框绑定一个键盘敲击事件等等。比如下面的代码，

```javascript
$('#button').on('click', function () {
    console.log('我被点了');
});
```

上面使用了JQuery的语法，给一个按钮绑定了一个事件。当事件触发时，会执行绑定的逻辑。这比较容易理解。

除了界面事件之外，通常我们还有各种网络请求事件，比如ajax，websocket等等。这些网络请求在不同阶段也会触发各种事件，如果程序中有绑定相关处理逻辑，那么当事件触发时就会去执行相关逻辑。

除此之外，我们还可以自定义事件。比如，

```javascript
$('#div').on('data-loaded', function () {
    console.log('data loaded');
});

$('#div').trigger('data-loaded');
```

上面采用JQuery的语法，我们自定义了一个事件，叫做"data-loaded"，并在此事件上定义了一个触发逻辑。当我们通过`trigger`触发这个事件时，之前绑定的逻辑就会执行了。

# 观察者模式

之前在事件监听中提到了*自定义事件*，其实自定义事件是观察者模式的一种具体表现。观察者模式，又称为消息订阅/发布模式。它的含义是，我们先假设有一个“信号中心”，当某个任务执行完毕就向信号中心发出一个信号（事件），然后信号中心收到这个信号之后将会进行广播。如果有其他任务订阅了该信号，那么这些任务就会收到一个通知，然后执行任务相关的逻辑。

下面是观察者模式的一个简单实现（可参阅[用AngularJS实现观察者模式](http://gejiawen.github.io/2014/07/18/observe-pattern-by-angularjs/)），

```javascript
var ob = {
    channels: [],
    subscribe: function(topic, callback) {
       if (!_.isArray(this.channels[topic])) {
           channels[topic] = [];
       }
       var handlers = channels[topic];
       handlers.push(callback);
    },
    unsubscribe: function(topic, callback) {
       if (!_.isArray(this.channels[topic])) {
           return;
       }
       var handlers = this.channels[topic];
       var index = _.indexOf(handlers, callback);
       if (index >= 0) {
           handlers.splice(index, 1);
       }
   },
   publish: function(topic, data) {
       var self = this;
       var handlers = this.channels[topic] || [];
       _.each(handlers, function(handler) {
           try {
               handler.apply(self, [data]);
           } catch (ex) {
               console.log(ex);
           }
       });
   }
};
```

其用法如下，

```javascript
ob.subscribe('done', function () {
    console.log('done');
});

setTimeout(function () {
    ob.publish('done')
}, 1000);
```

观察者模式的实现方式有很多，不过基本核心都差不多，都会有消息订阅和发布。从本质上说，前面所说的事件监听也是一种观察者模式。

观察者模式用好了自然好处多多，能够把解耦做的相当好。但是复杂的系统如果要用观察者模式来做逻辑，必须要做好事件订阅和发布的设计，否则会导致程序的运行流程混乱。

# Promise模式

Promise严格来说不是一种新技术，它只是一种语法糖，用于管理异步回调。Promise源自[Promises/A规范](http://wiki.commonjs.org/wiki/Promises/A)。使用promise来管理回调，可以将回调逻辑扁平化，可以避免之前提到的回调地狱。JQuery有一套Promise的实现，示例代码如下，

```javascript
function fn1() {
    var dfd = $.Deferred();
    setTimeout(function () {
        console.log('fn1');
        dfd.resolve();
    }, 1000);
    return dfd.promise();
}

function fn2() {
    console.log('fn2');
}

fn1().then(fn2);
```

这里稍微解释一下。Promise的本质其实是一个有限状态机，它共有三种状态：pending（执行中）、fulfilled（成功）、rejected（失败）。其中pending为初始装填，fulfilled和rejected是终止状态。状态只能从初始状态到终止状态转换，即pending->fulfilled或者pending->rejected。在状态转换的过程中会触发各种事件。

针对之前提到的回调地狱和异常难以捕获的问题，使用promise都可以轻松的解决。

```javascript
A().then(B).then(C).then(D).catch(ERROR);
```

看，一行就搞定了。不过使用promise处理异步调用，有一点需要注意，就是所有的异步函数都要**promise化**。所谓promise化的意思就是需要对异步函数进行封装，让其返回一个promise对象。比如，

```javascript
function A() {
    var promise = new Promise(function (resolve, reject) {
        // your logic 
    });
    
    return promise;
}
```














