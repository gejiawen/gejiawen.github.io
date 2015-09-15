postid: "action-about-angularjs-directives"
title: AngularJS指令间的交互
date: 2014-11-05 10:24:34
categories: [AngularJS]
tags: [ng, 实践]
---

本文将介绍AngularJS中指令间的交互方法。

假设我们有这样一个场景，

> 在一个html元素中有多条指令，且指令间有一些逻辑上的交互。

那么，我们如何来创建这些指令，而且让这些指令能够交互起来呢？

看下面的html代码，

```html
<superman strength>动感超人---力量</superman>
<superman strength speed>动感超人2---力量+敏捷</superman>
<superman strength speed light>动感超人3---力量+敏捷+发光</superman>
```

从代码中可以看出，上面html代码中包含了4个自定义的指令`superman`，`strength`，`speed`和`light`。

其中`superman`指令的代码如下，

```javascript
var myModule = angular.module("MyModule", []);
myModule.directive("superman", function() {
    return {
        scope: {},
        restrict: 'AE',
        controller: function($scope) {
            $scope.abilities = [];
            this.addStrength = function() {
                $scope.abilities.push("strength");
            };
            this.addSpeed = function() {
                $scope.abilities.push("speed");
            };
            this.addLight = function() {
                $scope.abilities.push("light");
            };
        },
        link: function(scope, element, attrs) {
            element.bind("mouseenter", function() {
                console.log(scope.abilities);
            });
        }
    }
});
```

`superman`指令中的`scope`属性表示该指令的独立作用域，指令的独立作用域其实是跟指令的实例绑定的，即不同的指令实例，其独立作用域是不同的。这是复用指令的必要条件。

`superman`指令中的`restrict`属性表示该指令的使用形式。有4个选项：**EAMC**。可以参考之前的[一篇文章](http://gejiawen.github.io/2014/07/16/AngularJS/AngularJS-Directive%E7%94%A8%E6%B3%95%E8%AF%B4%E6%98%8E/#return对象参数说明)，这里不多作解释。

`superman`指令中的`controller`属性是指令内部的控制器。其目的一般是为该指令暴露一些接口供外部使用。控制器函数的参数`$scope`就是指向该指令的独立作用域。

其他三个指令的代码如下，

```javascript
myModule.directive("strength", function() {
    return {
        require: '^superman',
        link: function(scope, element, attrs, supermanCtrl) {
            supermanCtrl.addStrength();
        }
    }
});
myModule.directive("speed", function() {
    return {
        require: '^superman',
        link: function(scope, element, attrs, supermanCtrl) {
            supermanCtrl.addSpeed();
        }
    }
});
myModule.directive("light", function() {
    return {
        require: '^superman',
        link: function(scope, element, attrs, supermanCtrl) {
            supermanCtrl.addLight();
        }
    }
});
```

这里需要注意的地方就是，`strength`，`speed`及`light`指令的定义中使用了`require`属性，这个`require`的解释如下，

> 请求另外的controller，传入当前directive的linking function中。require需要传入一个directive controller的名称。如果找不到这个名称对应的controller，那么将会抛出一个error。名称可以加入以下前缀：

> **`?`** 不要抛出异常。这将使得这个依赖变为一个可选项。

> **`^`** 允许查找父元素的controller。

在这里，这三个指令都依赖之前的`superman`指令。且在指令的`link`方法中使用了第四个参数`supermanCtrl`，这个参数其实就是指令`superman`的`controller`的注入。所以在这三个指令的`link`方法中可以使用`superman`指令控制器中定义的几个方法。

最开始的那一段html代码的运行效果就是，

- 鼠标划过文本时，会在console中输出`$scope.abilities`的值
- 因为`superman`指令中`abilities`属性其实是属于独立作用域的，所以html中每一行的`superman`都会有一个独立的`abilities`属性

总结：指令间的交互依赖指令的`controller`创建一些供外部调用的接口，而调用这些接口的指令需要`require`接口的指令名称。


