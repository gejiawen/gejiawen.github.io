title: 创建可复用angularjs组件
date: 2014-07-17 17:04:29
categories: [AngularJS]
tags: [ng]
---

AngularJS框架可以用`Service`和`Directive`降低开发复杂性。这个特性非常适合用于分离代码，创建可测试组件，然后将它们变成可重用组件。

`Directive`是一组独立的JavaScript、HTML和CSS，它们封装了一个特定的行为，它将成为将来创建的Web组件的组成部分，我们可以在各种应用中重用这些组件。在创建之后，我们可以直接通过一个HTML标签、自定义属性或CSS类、甚至可以是HTML注释，来执行一个Directive。

这一篇教程将介绍如何创建一个**自定义步长选择**Directive，它可以作为一个可重用输入组件。本文不仅会介绍Directive的一般创建过程，还会介绍输入控件验证方法，以及如何使用ngModelController无缝整合任意表单，从而利用AngularJS表单的现有强大功能。


下面直接放出相关的代码，

```html
<body ng-app="demo" ng-controller="DemoController">
    <form name="form" >
        Model value : <input type="text" size="3" ng-model="rating"><br>
        Min value: <input type="text" size="3" ng-model="minRating"><br>
        Max value: <input type="text" size="3" ng-model="maxRating"><br>
        Form has been modified : {{ form.$dirty }}<br>
        Form is valid : {{ form.$valid }}
        <hr>
        <div min="minRating" max="maxRating"  ng-model="rating" rn-stepper></div>
    </form>
</body>
```

```javascript
angular.module('demo', [
    'revolunet.stepper'
]) .controller('DemoController', function($scope) {
    $scope.rating = 42;
    $scope.minRating = 40;
    $scope.maxRating = 50;
});
```

# `rn-stepper`最简结构

```javascript
// we declare a module name for our projet, and its dependencies (none)
angular.module('revolunet.stepper', [])
// declare our naïve directive
.directive('rnStepper', function() {
    return {
        // can be used as attribute or element
        restrict: 'AE',
        // which markup this directive generates
        template:
            '<button>-</button>' +
            '<div>0</div>' +
            '<button>+</button>'
    };
});
```

现在directive rnStepper 已经有了一个简单的雏形了。可以有如下两种使用方法：
* `<div rn-stepper> </div>`
* `<rn-stepper> </rn-stepper>`

这里有一个[demo](http://jsfiddle.net/revolunet/n4JHg/)

# 添加内部动作

直接上代码，

```javascript
xxx.directive('rnStepper', function() {
    return {
        restrict: 'AE',
        // declare the directive scope as private (and empty)
        scope: {},
        // add behaviour to our buttons and use a variable value
        template:
                '<button ng-click="decrement()">-</button>' +
                '<div>{{value}}</div>' +
                '<button ng-click="increment()">+</button>',

        // this function is called on each rn-stepper instance initialisation
        // we just declare what we need in the above template
        link: function(scope, iElement, iAttrs) {
            scope.value = 0;
            scope.increment = function() {
                scope.value++;
            };
            scope.decrement = function() {
                scope.value--;
            };
        }
    };
});
```

我们在template中，分别给两个button添加了click事件响应，在link方法中实现了响应的方法。 这里的scope是一个**private scope**，其作用域仅限rnStepper这个directive。

这里有一个[demo](http://jsfiddle.net/revolunet/A92Aw/)


# 与外部世界（外部作用域）的交互

直到上面为止，我们的`rnStepper`都是自己跟自己玩，并没有跟外部作用域进行一些交互。下面我们将添加一个数据绑定，使rnStepper与外部世界建立联系。

我们给`scope`添加一个`value`属性，

```javascript
scope: {
    value: '=ngModel'
}
```

我们在scope中添加了一组键值对，这样，会自动建立内部变量value与外部属性ngModel的联系。 这里的**`=`**代表的意思是**双向绑定**（double data-binding）。

什么叫双向绑定? 即： 当`value`发生改变，那么`ngModel`也会发生改变，反之亦然。

在我们的这个demo中，看下面这行代码：

```html
<div rn-stepper ng-model="rating"></div>
```

这里的意思就是： directive `rnStepper`的内部变量`value`与外部`scope`中的`rating`建立了双向数据绑定。

这里有一个[demo](http://jsfiddle.net/revolunet/9e7Hy/)。


# 让我们组件更加友好

直接上代码，

```javascript
.directive('rnStepper', function() {
    return {
        // restrict and template attributes are the same as before.
        // we don't need anymore to bind the value to the external ngModel
        // as we require its controller and thus can access it directly
        scope: {},
        // the 'require' property says we need a ngModel attribute in the declaration.
        // this require makes a 4th argument available in the link function below
        require: 'ngModel',
        // the ngModelController attribute is an instance of an ngModelController
        // for our current ngModel.
        // if we had required multiple directives in the require attribute, this 4th
        // argument would give us an array of controllers.
        link: function(scope, iElement, iAttrs, ngModelController) {
            // we can now use our ngModelController builtin methods
            // that do the heavy-lifting for us

            // when model change, update our view (just update the div content)
            ngModelController.$render = function() {
                iElement.find('div').text(ngModelController.$viewValue);
            };

            // update the model then the view
            function updateModel(offset) {
                // call $parsers pipeline then update $modelValue
                ngModelController.$setViewValue(ngModelController.$viewValue + offset);
                // update the local view
                ngModelController.$render();
            }

            // update the value when user clicks the buttons
            scope.decrement = function() {
                updateModel(-1);
            };
            scope.increment = function() {
                updateModel(+1);
            };
        }
    };
});
```

这里，我不在需要内部变量value了。因为我们在link方法中已经拿到了`ngModelController`的引用，这里的`ngModelController.$viewValue`其实就是前面例子中的value。

同时我们又添加了另外一组键值对require: 'ngModel'。

我们使用了两个新的API：
* `ngModelController.$render`: 在ngModel发生改变的时候框架自动调用，同步`$modelValue`和`$viewValue`， 即刷新页面。
* `ngModelController.$setViewValue`： 当`$viewValue`发生改变时，通过此方法，同步更新`$modelValue`。

这里有一个[demo](http://jsfiddle.net/revolunet/s4gm6/)

End! All rights reserved `@gejiawen`.
