postid: "usage-for-angularjs-directive"
title: 'AngularJS Directive用法说明'
date: 2014-07-16 14:22:51
categories: [AngularJS]
tags: [ng]
---

**Directive**可能是AngularJS中比较复杂的一个东西了。一般我们将其理解成**指令**。AngularJS自带了不少预设的指令，比如`ng-app`，`ng-controller`这些。可以发现个特点，AngularJS自带的指令都是由`ng-`打头的。

那么，**Directive**究竟是个怎么样的一个东西呢？我个人的理解是这样的：*将一段html、js封装在一起，形成一个可复用的独立个体，具体特定的功能*。下面我们来详细解读一下**Directive**的一般性用法。


# AnguarJS directive的常用定义格式以及参数说明

## 直接return

看下面的代码：

```javascript
var myDirective = angular.module('directives', []);

myDirective.directive('directiveName', function($inject) {
    return {
        template: '<div></div>',
        replace: false,
        transclude: true,
        restrict: 'E',
        scope: {},
        controller: function($scope, $element) {

        },
        complie: function(tElement, tAttrs, transclude) {
            return {
                pre: function preLink(scope, iElement, iAttrs, controller) {

                },
                post: function postLink(scope, iElement, iAttrs, controller) {

                }
            };
        },
        link: function(scope, iElement, iAttrs) {

        }
    };
});
```

这里直接return了一个object，对象中包括比较多的属性，这些属性都是对自定义directive的定义。详细的含义，下面会继续说明。

## return对象参数说明

```javascript
return {
    name: '',
    priority: 0,
    terminal: true,
    scope: {},
    controller: fn,
    require: fn,
    restrict: '',
    template: '',
    templateUrl: '',
    replace: '',
    transclude: true,
    compile: fn,
    link: fn
}
```

如上所示，return的对象中会有很多的属性，这行属性都是用来定义directive的。下面我们来一个个的说明他们的作用。

* `name`
表示当前scope的名称，一般声明时使用默认值，不用手动设置此属性。
* `priority`
优先级。当有多个directive定义在同一个DOM元素上时，有时需要明确他们的执行顺序。这个属性用于在directive的compile function调用之前进行排序。如果优先级相同，则执行顺序是不确定的（根据经验，优先级高的先执行，相同优先级时按照**先绑定后执行**）。
* `teminal`
最后一组。如果设置为`true`，则表示当前的priority将会成为最后一组执行的directive，即比此directive的priority更低的directive将不会执行。同优先级依然会执行，但是顺序不确定。
* `scope`
    * `true`
        将为这个directive创建一个新的scope。如果在同一个元素中有多个directive需要新的scope的话，它还是只会创建一个scope。新的作用域规则不适用于根模版，因为根模版往往会获得一个新的scope。
    * `{}`
        将创建一个新的、独立的scope，此scope与一般的scope的区别在于**它不是通过原型继承于父scope的**。这对于创建可复用的组件是很有帮助的，可以有效的防止读取或者修改父级scope的数据。这个独立的scope会创建一个拥有一组来源于父scope的本地scope属性hash集合。这些本地scope属性对于模版创建值的别名很有帮助。本地的定义是对其来源的一组本地scope property的hash映射。
* `controller`
controller构造函数。controller会在pre-linking步骤之前进行初始化，并允许其他directive通过指定名称的require进行共享。这将允许directive之间相互沟通，增强相互之间的行为。controller默认注入了以下本地对象：
    * `$scope` 与当前元素结合的scope
    * `$element` 当前的元素
    * `$attrs`  当前元素的属性对象
    * `$transclude` 一个预先绑定到当前scope的转置linking function
* `require`
请求另外的controller，传入当前directive的linking function中。require需要传入一个directive controller的名称。如果找不到这个名称对应的controller，那么将会抛出一个error。名称可以加入以下前缀：
    * `?` 不要抛出异常。这将使得这个依赖变为一个可选项
    * `^` 允许查找父元素的controller
* `restrict`
EACM的子集的字符串，它限制了directive为指定的声明方式。如果省略的话，directive将仅仅允许通过属性声明
    * `E` 元素名称： <my-directive></my-directive>
    * `A` 属性名：<div my-directive="exp"></div>
    * `C` class名：<div class="my-directive:exp"></div>
    * `M` 注释：<!-- directive: my-directive:exp -->
* `template`
如果`replace`为true，则将模版内容替换当前的html元素，并将原来元素的属性、class一并转移；如果`replace`为false，则将模版元素当作当前元素的子元素处理。
* `templateUrl`
与template基本一致，但模版通过指定的url进行加载。因为模版加载是异步的，所有compilation、linking都会暂停，等待加载完毕后再执行。
* `replace`
如果设置为true，那么模版将会替换当前元素，而不是作为子元素添加到当前元素中。（**为true时，模版必须有一个根节点**）
* `transclude`
编译元素的内容，使它能够被directive使用。需要在模版中配合`ngTransclude`使用。transclusion的有点是linking function能够得到一个预先与当前scope绑定的transclusion function。一般地，建立一个widget，创建独立scope，transclusion不是子级的，而是独立scope的兄弟级。这将使得widget拥有私有的状态，transclusion会被绑定到父级scope中。**（上面那段话没看懂。但实际实验中，如果通过<any my-directive>{{name}}</any my-directive>调用myDirective，而transclude设置为true或者字符串且template中包含<sometag ng-transclude>的时候，将会将{{name}}的编译结果插入到sometag的内容中。如果any的内容没有被标签包裹，那么结果sometag中将会多了一个span。如果本来有其他东西包裹的话，将维持原状。但如果transclude设置为’element’的话，any的整体内容会出现在sometag中，且被p包裹）**
    * true/false  转换这个directive的内容。（这个感觉上，是直接将内容编译后搬入指定地方）
    * 'element'  转换整个元素，包括其他优先级较低的directive。（像将整体内容编译后，当作一个整体（外面再包裹p），插入到指定地方）
* `compile`
这里是compile function，将在下面实例详细说明
* `link`
这里是link function ，将在下面实例详细讲解。这个属性仅仅是在compile属性没有定义的情况下使用。

## 关于scope

这里关于directive的scope为一个object时，有更多的内容非常有必要说明一下。看下面的代码：

```javascript
scope: {
    name: '=',
    age: '=',
    sex: '@',
    say: '&'
}
```

这个`scope`中关于各种属性的配置出现了一些奇怪的前缀符号，有`=`，`@`，`&`，那么这些符号具体的含义是什么呢？再看下面的代码：

```html
<div my-directive name="myName" age="myAge" sex="male" say="say()"></div>
```

```javascript
function Controller($scope) {
    $scope.name = 'Pajjket';
    $scope.age = 99;
    $scope.sex = '我是男的';
    $scope.say = function() {
        alert('Hello，我是弹出消息');
    };
}
```

可以看出，几种修饰前缀符的大概含义：

* `=`: 指令中的属性取值为Controller中对应$scope上属性的取值
* `@`: 指令中的取值为html中的字面量/直接量
* `&`： 指令中的取值为Controller中对应$scope上的属性，但是这个属性必须为一个函数回调

下面是更加官方的解释：

* `=`或者`=expression/attr`
    在本地scope属性与parent scope属性之间设置双向的绑定。如果没有指定`attr`名称，那么本地名称将与属性名称一致。
    例如：
    <widget my-attr="parentModel">中，widget定义的scope为：`{localModel: '=myAttr'}`，那么widget scope property中的`localName`将会映射父scope的`parentModel`。如果`parentModel`发生任何改变，localModel也会发生改变，反之亦然。即**双向绑定**。
* `@`或者`@attr`
    建立一个local scope property到DOM属性的绑定。因为属性值总是String类型，所以这个值总返回一个字符串。如果没有通过`@attr`指定属性名称，那么本地名称将与DOM属性的名称一致。
    例如：
    <widget my-attr="hello {{name}}">，widget的scope定义为：`{localName: '@myAttr'}`。那么，widget scope property的localName会映射出`"hello {{name}}"`转换后的真实值。当name属性值发生改变后，widget scope的localName属性也会相应的改变（**仅仅是单向，与上面的`=`不同**）。那么属性是在父scope读取的（不是从组件的scope读取的）
* `&`或者`&attr`
    提供一个在父scope上下文中执行一个表达式的途径。如果没有指定attr的名称，那么local name将与属性名一致。
    例如：
    `<widget my-attr="count = count + value">`，widget的scope定义为：`{localFn:’increment()’}`，那么isolate scope property `localFn`会指向一个包裹着increment()表达式的function。
    一般来说，我们希望通过一个表达式，将数据从isolate scope传到parent scope中。这可以通过传送一个本地变量键值的映射到表达式的wrapper函数中来完成。例如，如果表达式是increment(amount)，那么我们可以通过`localFn({amount:22})`的方式调用localFn以指定amount的值。





# directive 实例讲解


下面的示例都围绕着上面所作的参数说明而展开的。

## directive声明实例

```javscript

// 自定义directive
var myDirective = angular.modeule('directives', []);

myDirective.directive('myTest', function() {
    return {
        restrict: 'EMAC',
        require: '^ngModel',
        scope: {
            ngModel: '='
        },
        template: '<div><h4>Weather for {{ngModel}}</h4</div>'
    };
})；

// 定义controller
var myControllers = angular.module('controllers', []);
myControllers.controller('testController', [
    '$scope',
    function($scope) {
        $scope.name = 'this is directive1';
    }
]);


var app = angular.module('testApp', [
    'directives',
    'controllers'
]);
```

```html
<body ng-app="testApp">
    <div ng-controller="testController">
        <input type="text" ng-model="city" placeholder="Enter a city" />
        <my-test ng-model="city" ></my-test>
        <span my-test="exp" ng-model="city"></span>
        <span ng-model="city"></span>
    </div>
</body>
```

## template与templateUrl的区别和联系

templateUrl其实根template功能是一样的，只不过templateUrl加载一个html文件，上例中，我们也能发现问题，template后面根的是html的标签，如果标签很多呢，那就比较不爽了。可以将上例中的，template改一下。

```javascript
myDirective.directive('myTest', function() {
    return {
        restrict: 'EMAC',
        require: '^ngModel',
        scope: {
            ngModel: '='
        },
        templateUrl:'../partials/tem1.html'   //tem1.html中的内容就是上例中template的内容。
    }
});
```

## scope重定义

```javascript
//directives.js中定义myAttr
myDirectives.directive('myAttr', function() {
    return {
        restrict: 'E',
        scope: {
            customerInfo: '=info'
        },
        template: 'Name: {{customerInfo.name}} Address: {{customerInfo.address}}<br>' +
                  'Name: {{vojta.name}} Address: {{vojta.address}}'
    };
});

//controller.js中定义attrtest
myControllers.controller('attrtest',['$scope',
    function($scope) {
        $scope.naomi = {
            name: 'Naomi',
            address: '1600 Amphitheatre'
        };
        $scope.vojta = {
            name: 'Vojta',
            address: '3456 Somewhere Else'
        };
    }
]);
```

```html
// html中
<body ng-app="testApp">
    <div ng-controller="attrtest">
        <my-attr info="naomi"></my-attr>
    </div>
</body>
```

其运行结果如下：

```shell
Name: Naomi Address: 1600 Amphitheatre      //有值,因为customerInfo定义过的
Name: Address:                              //没值 ，因为scope重定义后，vojta是没有定义的
```

我们将上面的directive简单的改一下，

```javascript
myDirectives.directive('myAttr', function() {
    return {
        restrict: 'E',
        template: 'Name: {{customerInfo.name}} Address: {{customerInfo.address}}<br>' +
                  'Name: {{vojta.name}} Address: {{vojta.address}}'
    };
});
```

运行结果如下：

```shell
Name: Address:
Name: Vojta Address: 3456 Somewhere Else
```

因为此时的directive没有定义独立的scope，customerInfo是undefined，所以结果正好与上面相反。

## transclude的使用

**transclude的用法，有点像jquery里面的$().html()功能**

```javascript
myDirective.directive('myEvent', function() {
    return {
        restrict: 'E',
        transclude: true,
        scope: {
            'close': '$onClick'      //根html中的on-click="hideDialog()"有关联关系
        },
        templateUrl: '../partials/event_part.html'
    };
});

myController.controller('eventTest', [
    '$scope',
    '$timeout',
    function($scope, $timeout) {
        $scope.name = 'Tobias';
        $scope.hideDialog = function() {
            $scope.dialogIsHidden = true;
            $timeout(function() {
                $scope.dialogIsHidden = false;
            }, 2000);
        };
    }
]);
```

```html
<body ng-app="phonecatApp">
    <div ng-controller="eventtest">
        <my-event ng-hide="dialogIsHidden" on-click="hideDialog()">
            Check out the contents, {{name}}!
        </my-event>
    </div>
</body>
```

```html
<!--event_part.html -->
<div>
    <a href ng-click="close()">×</a>
    <div ng-transclude></div>
</div>
```

说明：这段html最终的结构应该如下所示：

```html
<body ng-app="phonecatApp">
    <div ng-controller="eventtest">
        <div ng-hide="dialogIsHidden" on-click="hideDialog()">
            <span>Check out the contents, {{name}}!</span>
        </div>
    </div>
</body>
```

将原来的html元素中的元素`Check out the contents, {{name}}!`插入到模版的`<div ng-transclude></div>`中，还会另外附加一个`<span>`标签。

## controller，link，compile之间的关系

```javascript
myDirective.directive('exampleDirective', function() {
    return {
        restrict: 'E',
        template: '<p>Hello {{number}}!</p>',
        controller: function($scope, $element){
            $scope.number = $scope.number + "22222 ";
        },
        link: function(scope, el, attr) {
            scope.number = scope.number + "33333 ";
        },
        compile: function(element, attributes) {
            return {
                pre: function preLink(scope, element, attributes) {
                    scope.number = scope.number + "44444 ";
                },
                post: function postLink(scope, element, attributes) {
                    scope.number = scope.number + "55555 ";
                }
            };
        }
    }
});
```

```html
//controller.js添加
myController.controller('directive2',[
    '$scope',
    function($scope) {
        $scope.number = '1111 ';
    }
]);
```

```html
//html
<body ng-app="testApp">
    <div ng-controller="directive2">
        <example-directive></example-directive>
    </div>
</body>
```

上面小例子的运行结果如下：

```shell
Hello 1111 22222 44444 5555 !
```

**由结果可以看出来，controller先运行，compile后运行，link不运行。**

我们现在将`compile`属性注释掉后，得到的运行结果如下：

```shell
Hello 1111 22222 33333 !
```

**由结果可以看出来，controller先运行，link后运行，link和compile不兼容。一般地，compile比link的优先级要高。**
