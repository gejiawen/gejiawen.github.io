postid: "prevent-http-jitter"
title: http请求防抖动
date: 2014-10-30 10:15:03
categories: [祖传代码]
tags: [javascript, 实践]
---


假设我们正处在这样的一个场景：

> 程序将会根据我们的输入，实时的发送http请求，向后端请求数据。

此场景的一个实例如下：

有一个搜索框，我们在搜索框中输入一些关键字，程序将会**实时的**给出相关关键字的搜索结果。即所谓的即时搜索功能。

这里，可能会出现一个问题。当我们在输入框中连续的输入关键字时，每输入一个字符就会向后端发送一个http请求，这样就会导致**结果抖动**的效果。

假设我们的代码如下，

```html
<input type="text" ng-keyup="getResult()">
```

这里采用AngularJS的语法，为`input`标签绑定了一个键盘事件回调`getResult`，其内容如下：

```javascript
// more code

var ticket;
$scope.getResult = function() {
    if (ticket) {
        $timeout.cancel(ticket);
    }

    ticket = $timeout(function() {
        $http.get('/url')
        .success(function(data) {
            // do something
        })
        .error(function(data) {
            // do something
        });
    }, 350);
};

// more code
```

从代码中可以看出，每次触发键盘的按键弹起事件（即完成一次键盘输入）时，我们就会触发`getResult`回调。而在回调的内容中，我们采用`$timeout`服务来达到延时发出http请求的目的。**如果在延时的过程中（延时尚未结束），再次触发了`getResult`回调，那么`if`语句的判断将会取消上次的延时，即取消了之前的http请求。这样就达到了防御由于连续输入而造成的抖动问题。**

