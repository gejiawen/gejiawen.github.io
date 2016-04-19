postid: 'using-ng-model-controller-with-angular-directive'
title: 在Angular指令中使用NgModelController做数据绑定
categories: [AngularJS]
tags: [ng]
date: 2015-12-20 21:59:03

---

# 前言

AngularJS中的指令是其尤为复杂的一个部分，但是这也是其比较好玩的地方。今天我们就来说一说AngularJS中的[NgModelController](https://docs.angularjs.org/api/ng/type/ngModel.NgModelController)。

在AngularJS的内置指令中，有一个directive叫做[ngModel](https://docs.angularjs.org/api/ng/directive/ngModel)，我们可以用它来沟通控制器和视图层的数据交换。说的简单点，就是我们可以用它来做**双方数据绑定**。

这篇文章我们就来说一说如何在我们自定义的指令中，利用`ngModel`的`controller`来做双向数据绑定。

注意，本篇文章不是对`NgModelController`文档的说明，而是更偏向实践。下面我将全程带领大家去实现一个自定义指令，并且利用`ng-model`属性来做双方的数据绑定。

# 示例

我们的app中使用了一个自定义的指令，名字叫做`timeDruation`，如下，

```html
<div ng-app="HelloApp" ng-controller="HelloController">
    <h1>自定义指令</h1>
    <time-duration ng-model="test"></time-duration>
    <h1>默认指令</h1>
    <input ng-model="test">second
</div>
```

JS代码如下，

```javascript
angular.module('HelloApp', [])
    .directive('timeDuration', TimeDurationDirective);
    .controller('HelloController', function($scope) {
        $scope.test = 1;
    });
```

我们的示例指令可以做这样一件事，可以指定几个常见的时间单位，并且能够输入数据。最终我们将得到对应的秒数。其功能的截图如下，

![](/res/using-ng-model-controller-with-angular-directive/001.png)

这里我们特意将`test`变量分别绑定到我们的自定义指令和默认指令中，以观察其效果。

## 自定义指令

闲话少叙，show my code.

先上指令的模板。从上图中可以看出，指令包含一个输入框一个下拉选择框。

```html
<div class="time-duration">
    <input ng-model='num'>
    <select ng-model='unit'>
        <option value="seconds">Seconds</option>
        <option value="minutes">Minutes</option>
        <option value="hours">Hours</option>
        <option value="days">Days</option>
    </select>
</div>
```

模板其实很简单，这里就不多说了。下面我们来看看这个指令的逻辑部分。

```javascript
function TimeDurationDirective() {
    var tpl = '....'; // 指令模板代码就是上面的内容，这里就不复制了。
    
    return {
        restrict: 'E',
        replace: true,
        template: tpl,
        require: 'ngModel',
        scope: {},
        link: function(scope, element, attrs, ngModelController) {
            var multiplierMap = {
                seconds: 1,
                minutes: 60,
                hours: 3600,
                days: 86400
            };
            var multiplierTypes = ['seconds', 'minutes', 'hours', 'days'];

            // TODO
        }
    };
}
```

指令的`link`方法我们暂时TODO了它。后面会逐步完善。

我先来看看这个指令的定义，其中用到了`require`声明。我的博客中这篇文章[AngularJS Directive用法说明](http://blog.gejiawen.com/2014/07/16/usage-for-angularjs-directive/)对`require`的用法作了详细的说明。简单来说，`require`的作用就是为这个directive声明一个依赖关系，表明此directive依赖另一个指令的`controller`属性。

这里稍微说明一下require的衍生用法。我们可以在`require`前加上修辞量词，比如，

```javascript
return {
    require: '^ngModel'
}

return {
    require: '?ngModel'
}

return {
    require: '?^ngModel'
}
```

- `^`前缀修饰表示允许查找当前指令的父级指令，如果找不到对应指令的controller则抛出一个错误。
- `?`则表示将这个require动作变成一个可选项，意思就是找不到对应指令的controller就算了，不会抛出错误。
- 当然，我们也可以联合使用这两个前缀修饰。

相对`?ngModel`，`^ngModel`我们使用的频率要更加高一点。比如，

```html
<my-directive ng-model="my-model">
    <other-directive></other-directive>
</my-directive>
```

这时，我们在`other-directive`中使用`require: ^ngModel`，它将会自动查找`my-directive`指令声明中的`controller`属性。

## 使用NgModelController

当我们声明了`require: 'ngModel'`之后，在`link`方法中会注入第四个参数，这个参数就是我们require的那个指令对应的controller。这里就是内置指令`ngModel`的指控器`ngModeController`了。

```javascript
link: function (scope, element, attrs, ngModelCtrl) {
    // TODO
}
```

### `$viewValue`和`$modelValue`

在`ngModelController`中有两个很重要的属性，一个叫做`$viewValue`，一个叫做`$modeValue`。这两者的含义官方的解释如下，

> `$viewValue`: Actual string value in the view.
> `$modelValue`: The value in the model, that the control is bound to.

如果你对上面的官方解释有疑惑的话，我这里给出一种我个人的解释。

`$viewView`就是指令渲染模板所用的值，而`$modelView`是在控制器中流通的值。很多时候，这两个值可能是不一样的。

比如你在页面上展示了一个日期，它显示的可能是“Oct. 20 2015”这样的字符串，但是呢，这个字符串在控制器中对应的值可能是一个Javascript的Date对象的实例。

再比如，我们的这个`time-duration`示例中，`$viewValue`其实指的是指令模板中`num`和`unit`组合出来的值，而`$modelValue`是HelloAppController中`test`变量对应的值。

### `$formatters`和`$parses`

除了`$viewValue`和`$modelValue`这两个属性之外，还有两个用来处理他们的方法。分别是`$parses`和`$formatters`。

前者的是作用是将`$viewValue`->`$modelValue`，后者的作用恰好相反，是将`$modelValue`->`$viewValue`。

`time-duration`指令与外部控制器以及其内部的运作如下图，

![](/res/using-ng-model-controller-with-angular-directive/002.png)

1. 在外部控制器中（即这里的`HelloApp`的controller），我们通过`ng-model="test"`将test变量传入指令`time-duration`中，并建立绑定关系。
2. 在指令内部，`$modelValue`其实就是`test`值的一份拷贝。
3. 我们通过`$formatters()`方法将`$modelValue`转变成`$viewValue`。
4. 然后调用`$render()`方法将`$viewValue`渲染到directive template中。
5. 当我们通过某种途径监控到指令模板中的变量发生变化之后，我们调用`$setViewValue()`来更新`$viewValue`。
6. 与**（4）**相对应，我们通过`$parsers`方法将`$viewValue`转化成`$modelValue`。
7. 当`$modelValue`发生变化后，则会去更新HelloApp的UI。

### 完善指令逻辑

按照上面的流程，我们先来将`$modelValue`转化成`$viewValue`，然后在指令模板中进行渲染。

```javascript
// $formatters接受一个数组
// 数组是一系列方法，用于将modelValue转化成viewValue
ngModelController.$formatters.push(function(modelValue) {
    var unit = 'minutes', num = 0, i, unitName;
    modelValue = parseInt(modelValue || 0);
 
    for (i = multiplierTypes.length-1; i >= 0; i--) {
        unitName = multiplierTypes[i];

        if (modelValue % multiplierMap[unitName] === 0) {
            unit = unitName;
            break;
        }
    }
    
    if (modelValue) {
        num = modelValue / multiplierMap[unit];
    }

    return {
        unit: unit,
        num:  num
    };
});
```

最后返回的对象就是`$viewValue`的value。（当然`$viewValue`还会有其他的一些属性。）

第二步，我们调用`$render`方法将`$viewValue`渲染到指令模板中去。

```javascript
// $render用于将viewValue渲染到指令的模板中
ngModelController.$render = function() {
    scope.unit = ngModelCtrl.$viewValue.unit;
    scope.num = ngModelCtrl.$viewValue.num;
};
```

第三步，我们通过`$watch`来监控指令模板中`num`和`unit`变量。当其发生变化时，我们需要更新`$viewValue`。

```javascript
scope.$watch('unit + num', function() {
// $setViewValue用于更新viewValue
    ngModelController.$setViewValue({
        unit: scope.unit,
        num: scope.num
    });
});
```

第四步，我们通过`$parsers`将`$viewValue`->`$modelValue`。

```javascript
// $parsers接受一个数组
// 数组是一系列方法，用于将viewValue转化成modelValue
ngModelController.$parsers.push(function(viewValue) {
    var unit = viewValue.unit；
    var num = viewValue.num;
    var multiplier;

    multiplier = multiplierMap[unit];

    return num * multiplier;
});
```


这样一个双方的数据绑定逻辑就建立了。[这里](http://runjs.cn/detail/rjfiybzv)有一个在线示例，就是文中示例的完整代码，你可以去体验一下。


# 参考列表

- [官方文档：ngModel](https://docs.angularjs.org/api/ng/directive/ngModel)
- [官方文档：ngModelController](https://docs.angularjs.org/api/ng/type/ngModel.NgModelController)
- [stackoverflow](http://stackoverflow.com/questions/19383812/whats-the-difference-between-ngmodel-modelvalue-and-ngmodel-viewvalue)
- [Using NgModelController with custom directives](http://www.chroder.com/2014/02/01/using-ngmodelcontroller-with-custom-directives/)

