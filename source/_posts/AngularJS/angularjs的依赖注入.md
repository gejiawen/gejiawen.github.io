title: angularjs的依赖注入
date: 2014-07-17 17:37:14
categories: [AngularJS]
tags: [ng]
---

**AngularJS**为我们提供了`angular.injector(modules)`DI注入注射器。但是在我们使用注入的时候常常是不需要关心具体的如何注入。我们只需要按照其规则书写我们的angularjs代码就会很容易的得到angularjs的DI特性。


一般有三种方法来做依赖注入。

# 推断式注入

在angularjs中我们可以在我们需要注入的地方按照名称注入，这里要求参数名称必须和注入服务实例名称相同，一种名称约定。angularjs会提取参数名称查找相应DI实例注入。

```javascript
var myModule = angular.module('myModule', []);
myModule.factory('$alert', function($window) {
    return {
        alert: function(text) {
            $window.alert(text);
        }
    };
});
var myController = function($scope, $alert) {
    $scope.message = function(msg) {
        console.log(msg);
        $alert.alert(msg);
    };
};
myModule.controller("myController", myController);
```

# 标记注入

在angularjs中我们可以利用$inject标注DI注入,这里需要注入服务名称的顺序和构造参数名对应.这里可以解决以上约定的死板性。

```javascript
var myModule = angular.module('myModule', []);
myModule.factory('$alert', ['$window', function($window) {
    return {
        alert: function(text) {
            $window.alert(text);
        }
    };}]);

var myController = function($scope, $alert) {
    $scope.message = function(msg) {
        console.log(msg);
        $alert.alert(msg);
    };
};
myController.$inject = ['$scope', '$alert'];
myModule.controller("myController", myController);
```

在这里，

```javascript
var myController = function($scope, $alert) {
    $scope.message = function(msg) {
        console.log(msg);
        $alert.alert(msg);
    };
};
```

我们可以将`$scope`和`$alert`改为任意的参数名，但是`myController.$inject = ['$scope', '$alert'];`是**必须要有的**，且数组的元素与实际的服务名一致，顺序与controller中的参数顺序一致。

# 内联注入

对于directives，factory，filter等特殊指令使用`$inject`标注注入使用不是那么友好，angularjs特别增加了内联注入。如上面的`$alert`服务。

```javascript
myModule.factory('$alert', ['$window', function($window) {
   return {
       alert: function(text) {
       $window.alert(text);
    };
}]);
```

这里的`$window`就是采用内联注入的方式。同样的道理，在一些工厂方法中，比如controller、directive等，也可以采取这种方式来依赖注入。

```javascript
angular.module('app', [])
.controller('controllerName', [
    'dep1',
    'dep2',
    ...,
    function(dep1, dep2, ...) {
    ...
    }
]);
```

全文完。
