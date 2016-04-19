postid: 'some-javascript-async-pattern'
title: Javascript中常见的异步编程模型
categories: [Javascript]
tags: [javascript, Javascript异步编程专题]
date: 2015-10-12 15:26:30

---


Javascript异步编程专题，目前包含以下几篇文章，

- [浅谈Javascript中的异步](http://blog.gejiawen.com/2015/10/12/think-about-async-in-javascript/)
- [Javascript中常见的异步编程模型](http://blog.gejiawen.com/2015/10/12/some-javascript-async-pattern/)
- Generator函数处理异步调用

------

本篇文章是本专题的第二篇文章。

正文开始。

在Javascript异步编程专题的前一篇文章[浅谈Javascript中的异步](http://blog.gejiawen.com/2015/10/12/think-about-async-in-javascript/)中，我简明的阐述了“Javascript中的异步原理”、“Javascript如何在单线程上实现异步调用”以及“Javascript中的定时器”等相关问题。

本篇文章我将会谈一谈Javascript中常用的几种异步编程模型。

在前端的代码编写中，异步的场景随处可见。比如鼠标点击、键盘回车、网络请求等这些与浏览器紧密联系的操作，比如一些延迟交互特效等等。

在这些场景中，你必须要使用所谓的“异步模式”，否则将会严重程序的可行性和用户体验。我们列举这些场景中常用的几种异步编程模型，包括回调函数、事件监听、观察者模式（消息订阅/发布）、promise模式。除此之外还会稍微介绍一番ES6（ES7）中新增的方案。

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

## 回调函数的缺陷

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

除了上面这些问题之外，我觉得回调函数真正的核心问题在于，嵌套的回到函数往往会破坏整个程序的调用堆栈，并且像`return`，`throw`等这些用于代码流程控制的关键词都不能正常使用（因为前一个回调函数往往会影响到它后面所有的回调函数）。

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

下面是观察者模式的一个简单实现（可参阅[用AngularJS实现观察者模式](http://blog.gejiawen.com/2014/07/18/observe-pattern-by-angularjs/)），

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

Promise严格来说不是一种新技术，它只是一种语法糖，一种机制，一种代码结构和流程，用于管理异步回调。

jQuery中的Promise实现源自[Promises/A规范](http://wiki.commonjs.org/wiki/Promises/A)。使用promise来管理回调，可以将回调逻辑扁平化，可以避免之前提到的回调地狱。示例代码如下，

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

# ES6中的方案

ES6于今年6月份左右已经正式发布了。其中新增了不少内容。其中有两项内容可能用来解决异步回调的内容。

## ES6中的Promise

最新发布的ECMAScript2015中已经涵盖了promise的相关内容，不过ES6中的Promise规范其实是[Promise/A+规范](https://promisesaplus.com/)，可以说它是Promise/A规范的增强版。

现代浏览器Chrome，Firefox等已经对Promise提供了原生支持。详细的文档可以参阅[MDN](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise)。

简单来说，ES6中promise的内容具体如下，

- promise有三种状态：pending（等待）、fulfilled（成功）、rejected（失败）。其中pending为初始状态。
- promise的状态转换只能是：pending->fulfilled或者pending->rejected。转换方向不能颠倒，且fulfilled和rejected状态不能相互转换。每一种状态转换都会触发相关调用。
- pending->fulfilled时，promise会带有一个value（成功状态的值）；pending->rejected时，promise会带有一个reason（失败状态的原因）
- promise拥有`then`方法。`then`方法必须返回一个promise。`then`可以多次链式调用，且回调的顺序跟`then`的声明顺序一致。
- `then`方法接受两个参数，分别是“pending->fulfilled”的调用和“pending->rejected”的调用。
- `then`还可以接受一个promise实例，也可以接受一个thenable（类then对象或者方法）实例。


总得来说promise的内容比较简单，涉及到三种状态和两种状态转换。其实promise的核心就是`then`方法的实现。

下面是来自MDN上Promise的代码示例（稍作改动），

```javascript
var p1 = new Promise(function (resolve, reject) {
    console.log('p1 start');
    setTimeout(function() {
        resolve('p1 resolved');
    }, 2000);
});

p1.then(function (value) {
    console.log(value);
}, function(reason) {
    console.log(reason);
});
```

上述代码的执行结果是，先打印"p1 start"然后经过2秒左右再次打印"p1 resolved"。

当然我们还可以添加多个回调。我们可以通过在前一个`then`方法中调用`return`将promise往后传递。比如，

```javascript
p1.then(function(v) {
    console.log('1: ', v);
    return v + ' 2';
}).then(function(v) {
    console.log('2: ', v);
});
```

不过在使用Promise的时候，有一些需要注意的地方，这篇文章[We have a problem with promises](http://pouchdb.com/2015/05/18/we-have-a-problem-with-promises.html)（[翻译文](http://fex.baidu.com/blog/2015/07/we-have-a-problem-with-promises/)）中总结得很好，有兴趣的可自行参阅。

不管是ES6中的promise还是jQuery中的promise/deferred，的确可以避免异步代码的嵌套问题，使整体代码结构变得清晰，不用再受callback hell折磨。但是也仅仅止步于此，因为它并没有触碰js异步回调真正核心的内容。

现在业界有许多关于PromiseA+规范的实现，不过博主个人觉得[bluebird](https://github.com/petkaantonov/bluebird)是个不错的库，可以值得一用，如果你有选择困难症，不妨试一试😎😎😎

## ES6中Generator

ES6中引入的Generator可以理解为一种协程的实现机制，它允许函数在运行过程中将Javascript执行权交给其他函数（代码），并在需要的时候返回继续执行。

我们可以使用Generator配合ES6中Promise，进一步将异步调用扁平化（转化成同步风格）。

下面我们来看一个例子,

```javascript
function* gen() {
    var ret = yield new Promise(function(resolve, reject) {
        console.log('async task start');
        setTimeout(function() {
            resolve('async task end');
        }, 2000);
    });
    
    console.log(ret);
}
```

上述Node.js代码中，我们定义了一个Generator函数，且创建了一个promise，promise内使用`setTimeout`模拟了一个异步任务。

接下来我们来执行这个Generator函数，因为`yield`返回的是一个promise，所以我们需要使用`then`方法，

```javascript
var g = gen();
var result = g.next();

result.value.then(function(str){
    console.log(str);
    // 对resolve的数据重新包装，然后传递给下一个promise
    return {
        msg: str
    };
}).then(function(data){
    g.next(data);
});
```

最终的结果如下，

```bash
async task start
// 经过2秒左右
async task end
{msg: 'async task end'}
```

其实关于Generator还有很多的内容可以说，这里由于篇幅的关系就不展开了。业界已经有了基于Generator处理异步调用的功能库，比如[co](https://github.com/tj/co)、[task.js](http://taskjs.org/)。

## ES7中的async和await

在单线程的Javascript上做异步任务（甚至并发任务）的确是一个让人头疼的问题，总会越到各种各样的问题。从最早的函数回调，到Promise，再到Generator，涌现的各种解决方案，虽然都有所改进，但是仍然让人觉得并没有彻底的解决这个问题。

举个例子来说，我现在就是想读取一个文件，这么简单的一件事，何必要考虑那么多呢？又是回调，又是promise的，烦不烦呐。我就想像下面这么简单的写代码，难道不行么？

```javascript
function task() {
    var file1Content = readFile('file1path');
    var file2Content = readFile(fileContent);
    console.log(file2Content);
}
```

想要做的事情很简单，读取第一个文件，它的内容是要读取的第二个文件的文件名。

值得庆幸的是，ES7中的`async`和`await`可以帮你做到这件事。不过要稍微改动一下，

```javascript
async function task() {
    var file1Content = await readFile('file1path');
    var file2Content = await readFile(fileContent);
    console.log(file2Content);
}
```

看，改动的地方很简单，只要在`task`前面加上关键词`async`，在函数内的异步任务前添加`await`声明即可。如果忽略这些额外的关键字，简直就是完完全全的同步写法嘛。

其实，这种方式就是前端提到的Generator和Promise方案的封装。ECMAScript组织也认为这是目前解决Javascript异步回调的最佳方案，所以可能会在ES7中将其纳入到规范中来。需要注意的是，这项特性是ES7的提案，依赖Generator，所以慎用（目前来说基本用不了）！

## fibjs

除了上述的几种方案之外，其实还有另外一种方案。就是使用协程的方案来解决单线程上的异步调用问题。

之前我们也提到过，Generator的`yield`可以暂停函数执行，将执行权临时转交给其他任务，待其他任务完毕之后，再交还回执行权。这其实就是协程的基本模型。

业界有一款基于V8引擎的服务端开发框架[fibjs](http://fibjs.org)，它的实现机制跟Node.js是不一样的。fibjs采用[fiber](https://en.wikipedia.org/wiki/Fiber)解决v8引擎的多路复用，并通过大量c++组件，将重负荷运算委托给后台线程，释放v8线程，争取更大的并发时间。

一句话，fibjs从底层，使用的纤程模型解决了异步调用的问题。关于fibjs，有兴趣的话可以查阅相关资料。不过我个人对它是持谨慎态度的。原因是如下两点，

- 生态原因。
- 使用了js，但是又摒弃了js的异步。

不过还是可以作为兴趣去研究一下的。













