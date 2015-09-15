postid: "different-from-angularjs-provider-factory-service"
title: AngularJS服务中的provider、factory、service
date: 2014-07-14 16:17:18
categories: [AngularJS]
tags: [ng]
---

当我们创建一个AngularJS服务时，通常可以有三种方式来自定义service。

简单来说，`provider`是被 **注入控制器** 使用的一个对象，注入机制通过调用一个`provider`的`$get()`方法，把得到的东西作为参数进行相关调用。比如把得到的服务作为一个`Controller`的参数。

看下面的代码：

```javascript
// 这是一个provider
var pp = function() {
    this.$get = function() {
        return {
            'name': 'iceiceice',
            'getAge': function() {
                return 25;
            }
        };
    };
};

// 我在模块的初始化过程中，定义了一个叫 PP 的服务
var app = angular.module('Demo', [], function($provide) {
    $provide.provider('PP', pp);
});

// PP服务实际上就是 pp 这个provider的$get()方法返回的东西
app.controller('TestCtrl', function($scope, PP) {
    console.log(PP);
});
```


下面还有两种定义**service** 的简略方法。


第一个是`factory`方法，此方法由`$provider`提供，`module`的`factory`是一个引用，作用一样。这个方法直接把一个函数当成一个对象的`$get()`方法，这样你就不用显示的定义一个`provider`了。示例如下：

```javascript
var app = angular.module('Demo', [], function() {
    // TODO 模块声明
});

app.factory('PP', function() {
    return {
        'name': 'iceiceice'
    };
});

app.controller('TestCtrl', function($scope, PP) {
    console.log(PP);
});
```


第二个是`service`方法，也是由`$provider`提供，`module`中有对它的同门引用。

`service`和`factory`的区别在于：前者要求提供一个 **构造方法** ，后者是要求提供 `$get()`方法。简单的是说，就是前者一定是得到一个`object`， 后者可以一个数字或者字符串。示例代码如下：

```javascript
var app = angular.module('Demo', [], fyunction() {
    // TODO 模块声明
});

app.service('PP', function() {
    this.name = 'iceiceice';
});

app.controller('TestCtrl', function($scope, PP) {
    console.log(PP);
});
```


完整示例如下，

```html
<html ng-app="app">
    <head>
        <script src="jquery-2.0.3.js"></script>
        <script src="angular.js"></script>
    </head>
    <body ng-controller="ctrl">
    </body>

    <script>
        var app = angular.module('app', []);

        app.factory('PP1', function() {
            return {
                'name': 'jarred'
            };
        });

        app.service('PP2', function() {
            this.name = 'jarred2';
        });

        app.controller('ctrl', [
            '$scope',
            'PP1',
            'PP2',
            function($scope, pp1, pp2) {
                console.log(pp1);
                console.log(pp2);
            }
        ]);
    </script>
</html>
```
