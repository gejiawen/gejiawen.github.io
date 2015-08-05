title: 使用ES6的Generator代替回调函数
date: 2014-10-16 12:14:46
categories: [Javascript]
tags: [翻译, es6]
---

英文原文： [Replacing callbacks with ES6 Generators](http://modernweb.com/2014/02/10/replacing-callbacks-with-es6-generators/)

![](http://7xkwt1.com1.z0.glb.clouddn.com/使用ES6的Generator代替回调函数-001.jpg)

目前，已经有很多文章讨论过了如何使用[ES6 generators](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Iterators_and_Generators#Generators.3A_a_better_way_to_build_Iterators)来取代JavaScript中经常遇到的**回调金字塔**。但是，其中提到的绝大多数方法都需要依赖于某个库，而对于其中的原理却提及甚少。

在本文中，我们将一步一步的将一个基于回调函数的例子修改为一个基于generator的例子。本文的目标是让你透彻地理解使用generator替代回调函数的原理。

Generator是JavaScript中一个新概念，但在编程语言中已经存在已久。你可能已经在其他的编程语言例如Python使用过它。如果没有，也不要害怕，我们在后面已经为你准备了一个简单明了的入门介绍。

# 如何运行例子

在我们开始之前，你需要安装`Node 0.11.*`来运行文章中的例子。当你在运行这些例子时，你需要告诉Node使用ES6（也就是Harmony）来运行：`node -harmony example.js`。

# 什么是generator？

在我们深入讲述如何使用generator替代回调函数之前，我们先来说说什么是generator。

Generator很像是一个函数，但是你可以暂停它的执行。你可以向它请求一个值，于是它为你提供了一个值，但是余下的函数不会自动向下执行直到你再次向它请求一个值。

取号机也许是对generator的一个绝佳的比喻。你可以通过取一张票来向机器请求一个号码。你接收了你的号码，但是机器不会自动为你提供下一个。换句话说，取票机*暂停*直到有人请求另一个号码，此时它才会向后运行。

## ES6中的generator

Generator在ES6中像一个函数一样被声明，除了在之前有一个`*`的差别外：

```javascript
function* ticketGenerator() {

}
```

当你想要一个generator提供一个值然后暂停时，你需要使用`yield`关键字。`yield`有点像是`return`关键字，因为它们都返回一个值，但是函数在`yield`之后会进入暂停状态。

```javascript
function* ticketGenerator() {
    yield 1;
    yield 2;
    yield 3;
}
```

在我们的例子中，我们定义了一个叫做`ticketGenerator`的迭代器。如果你向它请求一个值，它会返回1然后暂停。如果你再次向它发出一个请求，我们将得到2，然后是3。

当你调用一个generator时，它将返回一个迭代器对象。这个迭代器对象拥有一个叫做`next`的方法来帮助你重启generator函数并得到下一个值。

`next`方法不仅返回值，它返回的对象具有两个属性：`done`和`value`。`value`是你获得的值，`done`用来表明你的generator是否已经停止提供值。

现在我们从我们的取号机中取一些号码，

```javascript
var takeANumber = ticketGenerator();
takeANumber.next();
// > { value: 1, done: false }
takeANumber.next();
// > { value: 2, done: false }
takeANumber.next();
// > { value: 3, done: false }
takeANumber.next();
// > { value: undefined, done: true }
```

现在我们的取号系统只能提供最多到3的号码，这实在是没什么用。我们想要让它无线增加下去，因此我们来创建一个循环。

```javascript
function* ticketGenerator() {
    for(var i=0; true; i++) {
        yield i;
    }
}
```

现在，如果这是一个普通的函数，我们每次只会得到0。但是使用generator却不一样：

```javascript
var takeANumber = ticketGenerator();
console.log(takeANumber.next().value); //0
console.log(takeANumber.next().value); //1
console.log(takeANumber.next().value); //2
console.log(takeANumber.next().value); //3
console.log(takeANumber.next().value); //4
```

每一次当我们调用`next()`时，generator执行下一个循环迭代然后暂停。这意味着我们拥有一个可以无限向下运行的generator。因为这个generator只是发生了暂停，你并没有冻结你的程序。事实上，generator是一个创建无限循环的好方法。

## 影响generator的状态

进一步探究迭代迭代generator对象的话，`next()`实际上还有另一个用途。如果你给`next`传递一个值，它会被视为generator中的一个`yield`语句的结果来对待。

因此`next`是一个在generator运行过程中向其传递信息的方式。我们将以此来修改我们的取号generator以便它能够被重置到0。我们希望能在任何时间点来重置取号机。

```javascript
function* ticketGenerator() {
    for(var i=0; true; i++) {
        var reset = yield i;
        if(reset) {
            i = -1;
        }
    }
}
```

正如你所看到的，如果`yield`返回了一个`true`然后我们将`i`设置为`-1`。那么`for`循环将会在循环的结尾将`i`增加`1`，因此下一次返回的`i`变成了`0`。

我们来看看实际情况如何：

```javascript
var takeANumber = ticketGenerator();
console.log(takeANumber.next().value); //0
console.log(takeANumber.next().value); //1
console.log(takeANumber.next().value); //2
console.log(takeANumber.next(true).value); //0
console.log(takeANumber.next().value); //1
```

# 用generator替代回调函数

既然我们已经学到了一些关于generator的知识，现在让我们来谈谈generator和回调函数。正如你所知道的，当我们调用例如AJAX请求这样的异步代码时我们会使用回调函数。为了简单起见我们在例子中定义一个delay函数。

我们的delay函数将会是异步的 – 在指定的时间过后我们提供给delay的回调函数才会被执行，然后delay会给你的回调函数传递一个字符串告诉它究竟**沉睡**了多久。

在此期间你的其余代码将会继续执行下去。这就好像是进行一个AJAX请求一样 – 你发出请求，你的代码继续执行，当服务器返回一个结果时你的回调函数才执行。

现在，我们来定义delay函数，

```javascript
function delay(time, callback){
    setTimeout(function(){
        callback("Slept for "+time);
    },time);
}
```

到目前为止，还没有什么特别的东西。现在我们来使用它来delay两次。首先我们将*delay 1000ms*，然后当delay结束后我们再另外*delay 1200ms*。

```javascript
delay(1000,function(msg){
    console.log(msg);
    delay(1200,function(msg){
        console.log(msg);
    });
});

//...waits 1000ms
// > "Slept for 1000"
//...waits another 1200ms
// > "Slept for 1200
```

确保我们的两个delay依次被调用的唯一方法就是确保第二个delay在第一个delay的回调函数中。

如果我们要依次delay 12次，我们将需要嵌套的调用12次delay函数。这时你就会碰到**回调金字塔**，代码也变得丑陋不堪。

## 引入generator

Generator是解决*回调地狱*的有效方法之一。异步调用是很困难的事情，因为我们的函数不会等待异步调用完成，因此我们需要回调函数。

使用generator，我们可以让我们的代码进行等待。无需嵌套回调函数，我们可以使用generator确保当异步调用在我们的generator函数运行一下行代码之前完成时暂停函数的执行。

因此，如果我们可以在一个异步调用完成时暂停执行，这就意味着我们可以依次调用delay函数 – 就像delay函数是同步执行的一样。

## 我们应该怎么做

首先，我们知道我们进行异步调用的代码需要在一个generator而不是一个一般的函数中进行，因此我们来定义一个generator，

```javascript
function* myDelayedMessages() {
    /* delay 1000 ms and print the result */
    /* delay 1200 ms and print the result */
}
```

接下来我们需要在我们的generator中调用delay。记住，delay接收一个回调函数。这个回调函数需要继续我们的generator，但是我们现在还没有一个generator因此我们先放上一个空函数。

```javascript
function* myDelayedMessages() {
    console.log(delay(1000, function(){}));
    console.log(delay(1200, function(){}));
}
```

我们代码依然是异步的。这是因为我们还没有将放入任何的yield语句。Generator只是在它们看大一个yield语句时才暂停。

```javascript
function* myDelayedMessages() {
    console.log(yield delay(1000, function(){}));
    console.log(yield delay(1200, function(){}));
}
```

我们现在已经更接近了一点了。然而，如果我们运行我们的generator什么也不会发生。因为没有什么东西告诉它要向下运行。

在这里你需要理解的最重要的概念是：generator需要在delay中的回调函数运行完成后继续往下运行，这就是它们如何知道暂停应该结束了的原因。

这意味着回调函数中的东西需要知道如何向前推动generator。我们在其中传递一个叫做`resume`的函数来为我们做这件事。记住我们现在还依然没有定义`resume`。

```javascript
function* myDelayedMessages(resume) {
    console.log(yield delay(1000, resume));
    console.log(yield delay(1200, resume));
}
```

OK，现在我们的generator将会接收一个resume函数，这个函数将会向前推动generator。

现在到了关键步骤了，我们如何来编写`resume`，它又是怎么来了解我们的generator的。

如果你看看其他使用generator代替回调函数的例子，你会看到generator函数总是被另一个函数包裹着 – 通常是一个叫做`run`或者`execute`的函数。这些函数的目的有以下几个:

- 接收一个generator作为参数
- 使用这个generator来创建一个新的generator迭代器对象，我们将调用它的next方法
- 创建一个`resume`函数来使用这个generator迭代器对象来推进generator
- 将`resume`函数传递给这个generator以便generator能够访问resume
- 在最开始时调用next()函数，以便我们的代码在碰到第一个`yield`之前开始执行

现在我们来创建`run`函数，

```javascript
function run(generatorFunction) {
    var generatorItr = generatorFunction(resume);

    function resume(callbackValue) {
        generatorItr.next(callbackValue);
    }

    generatorItr.next()
}
```

现在我们有了一个能够接收一个generator函数的函数，并为它传递了一个了解如何推进generator迭代器对象的函数。

注意到我们在`resume`函数中用到了`next`的第二个特性。`resume`是被传递给delay的回调函数，因此它接收delay函数提供的值。resume将这个值传递给next，因此yield语句的结果实际上是我们异步函数的结果！

我们现在要做的只是用`run`包裹上我们的generator函数，然后我们就能看到以下结果，

```javascript
run(function* myDelayedMessages(resume) {
    console.log(yield delay(1000, resume));
    console.log(yield delay(1200, resume));
});
//...waits 1000ms
// > "Slept for 1000"
//...waits 1200ms
// > "Slept for 1200"
```

现在，你能看到我们调用`delay`两次，并没有使用嵌套回调函数。如果你依然看到疑惑，我们现在概括的来讲述以下究竟发生了什么：

- `run`接收了我们的generator并创建了一个resume函数
- `run`创建了一个generator迭代器对象（我们在它上面调用`next`方法），提供了`resume`函数。接着它推动了generator迭代器向前运行。
- 我们的generator碰到了第一个yield语句并且调用delay。接着这个generator暂停。
- `delay`在1000ms之后完成然后调用`resume`。
- `resume`告诉我们的generator进行下一步。它将结果传递给delay以便`consol`e能够将它打印出来。
- 我们的generator碰到了第二个`yield`，调用`delay`然后再次暂停。 `delay`等待1200ms之后调用`resume`回调函数。
- `resume`再次推进generator。
- 再也没有`yield`的调用，这个generator完成执行。

# 结论

我们已经成功的使用generator替代了回调嵌套方法。总结一下，使用generator替代回调函数要包含以下几个步骤：

- 创建一个`run`函数来接受一个generator，并为这个generator提供resume函数。
- 创建一个`resume`函数来推进generator，然后在`resume`被异步函数调用时将这个`resume`函数传递给generator。
- 将`resume`作为回调传递给我们所有的异步回调函数。这些异步函数在完成时执行`resume`，这使得我们的generator在每个异步调用完成之时仅仅向前一步。

虽然generator究竟是不是一个处理*回调地狱*的好方法还在讨论之中，但是它确实是一个加强你对ES6中generator和迭代器理解的练习。如果你在寻找一些不需要用到ES6的处理嵌套回调函数的方法，可以考虑`promises`。

End. All rights reserved `@gejiawen`.


