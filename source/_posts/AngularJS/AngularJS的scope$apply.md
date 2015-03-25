title: AngularJS的scope.$apply
date: 2014-07-14 18:12:57
categories: [AngularJS]
tags: [ng]
---

如果你写过angular程序，肯定对`scope.$apply`不会陌生，表面上，他的作用就是把改变同步绑定到界面上。但是它为什么存在呢？我们什么时候需要用到它呢？什么时候不需要呢？

要真正理解$apply, 就必须知道我们为什么需要它。

首先，javascript是单线程执行的，我们写的javascript代码不是一口气就执行完的，而且是很多周期中完成的。每一个周期从开始到结束是不会被中断的，当浏览器主线程的一个周期在执行一段代码时，其他的代码就不会执行，UI渲染进程也会被挂起，浏览器就不会干被的事，处于一中冻结状态，所以糟糕的javascript能将浏览器挂起。这就是为什么《高性能javascript》书中说到的“一个函数执行的时间绝对不能超过100ms”的原因，当然`WebWorker`另当别论，`WebWorker`是浏览器主线程之外的一个线程，它不是阻塞的，但`WebWorker`是不能操作dom节点的。

当运行耗时的周期时，像ajax请求，等待点击事件，或者是设置延时，通过设置回调函数结束当前的运行的周期，网络请求，用户输入，定时等会交给底层的操作系统，当Ajax请求完成时，点击被触发，或者是延时结束时，一个新的javascript运行周期被创建，回调函数的内容被执行，这个过程和操作系统底层 cpu进程的运行逻辑是一致的。

上面所说的周期在javascript里就是Event Loop，对应了一个事件队列，回调事件，用户输入事件等都会放入队列。dom渲染也是事件驱动的，当然也就有渲染事件，现在大部分浏览器的渲染事件队列和javascript事件队列使用的是同一个事件队列，这就意味着渲染和执行是不会同时进行的。


# AngularJS 如何更新绑定？

angular的模板引擎允许我们把变量绑定到html模板中，而且还做到双向的绑定，它是如何做到的呢？angular怎么知道我们的变量改变了？angular又是怎么知道何时应该更新dom的呢？

先来说说，angular是怎么知道变量发生了改变。

要知道一个变量变了，方法不外乎两种：
* 第一种，只能通过固定的接口才能改变变量的值，比如说只能通过 set() 设置变量的值，set被调用时比较一下就知道了。这中方法的缺点洗是写法繁琐，只能用obj.set('key', 'value') 替代obj.key = 'value'。**EmberJS 和 KnockoutJS 都使用的是这种策略**
* 第二种, 脏检查，将原对象复制一份快照，在某个时间，比较现在对象与快照的值，如果不一样就表明发生变化。很明显，这个策略要保留两份变量，而且要遍历对象，比较每个属性，这样不会有性能问题吗？但偏偏angular使用的是这样方式，是google的程序员傻吗？angular没有性能问题吗？它是怎么做的呢？

我们先说这个好处，好处其实是给我们了，我们的任何对象都可以绑定，而且是可以随意赋值。

## 首先是脏检查的对象

angualar不会脏检查所有的对象，当对象被绑定到html中，这个对象添加为检查对象（watcher）。

angular不会脏检查所有的属性，同样当属性被绑定后，这个属性会被列为检查的属性。

我们可以结合源代码，看看大概的过程。下面是watcher的定义：

```javascript
watcher = {
    fn: listener,          //监听回调函数
    last: initWatchVal,    //上一状态值
    get: get,              //取得监听的值
    exp: watchExp,         //监听表达式
    eq: !!objectEquality   //要不要比较引用
};
```

在angular程序初始化时，会将绑定的对象的属性添加为监听对象（watcher），也就是说一个对象绑定了N个属性，就会添加N个watcher。

## 其次就是什么时候去脏检查

angular在我们所写的绝大部分代码中都会触发比较事件。比如：`controller`初始化的时候，`ng-click`事件和所有以`ng-`开头的事件执行后，`$http`回调完成后，都会触发脏检查。

当然，触发脏检查的点实在函数执行完之后，但不表明异步调用也执行完成，所以，如果我们的功能是异步的，那你会发现我们的改变并没有更新到dom上。

来个demo？好，请看：

```javascript
function Ctrl($scope) {
    $scope.message = "Waiting 2000ms for update";
    setTimeout(function () {
        $scope.message = "Timeout called!";
        // AngularJS unaware of update to $scope
    }, 2000);
}
```

dom上显示的message是 “Waiting 2000ms for update，**永远都不是 “Timeout called!”**

这就是`$apply`的应用场景，使用`$scope.$apply()`手动触发脏检查。其实angular还贴心的提供了一个`$timeout`，那`$timeout`和`setTimeout`在功能上有什么区别吗？可以说没有，唯一的区别就是，使用`$timeout`异步完成之后，angular会自动触发`$apply()`。

如果已经在一个具有`$apply`环境中，调用`$apply()`会抛出异常， 有空你可以试一试。

所以，如果我们的执行代码不在`$apply`环境中，比如我们查询sqlite的记录，结构是异步返回的，就需要用手动触发`$apply()`。

`$apply()`接受一个function的参数，function中被绑定的对象会被脏检查，function不能是异步的哦，这个很好理解，如果让你去实现这个apply函数，$apply()不带参数时，会将当前作用域的所有监听对象都脏检查一遍，所以不带参数是有害的, 必然会做很多无用的脏检查，浪费性能。


## 再次就是如何脏检查

脏检查只要是由`$digest()`完成，`$apply()`被调用之后，最终会触发`$digest()`，$digest 主要代码如下：

```javascript
if ((watchers = current.$$watchers)) {
    // process our watches
    length = watchers.length;

    while (length--) {
        try {
            watch = watchers[length];
            // Most common watches are on primitives, in which case we can short
            // circuit it with === operator, only when === fails do we use .equals
            if (watch
                && (value = watch.get(current)) !== (last = watch.last)
                && !(watch.eq ? equals(value, last) : (typeof value == 'number'
                    && typeof last == 'number' && isNaN(value) && isNaN(last)))) {
                dirty = true;
                watch.last = watch.eq ? copy(value) : value;
                watch.fn(value, ((last === initWatchVal) ? value : last), current);

                if (ttl < 5) {
                    logIdx = 4 - ttl;

                    if (!watchLog[logIdx]) {
                        watchLog[logIdx] = [];
                    }

                    logMsg = (isFunction(watch.exp)) ? 'fn: ' + (watch.exp.name || watch.exp.toString()) : watch.exp;
                    logMsg += '; newVal: ' + toJson(value) + ';oldVal: ' + toJson(last);
                    watchLog[logIdx].push(logMsg);
                }
            }
        } catch (e) {
            $exceptionHandler(e);
        }
    }
}
```

遍历watchers，比较监视的属性是否变化。

End! All rights reserved `@gejiawen`.
