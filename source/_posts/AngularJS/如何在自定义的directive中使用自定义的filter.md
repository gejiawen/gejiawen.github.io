postid: "how-to-use-custom-filter-in-custom-directive"
title: 如何在自定义的directive中使用自定义的filter
date: 2014-10-24 15:49:40
categories: [AngularJS]
tags: [ng, 实践]
---

笔者前两天在使用AngularJS做一个项目时，遇到这样一个需求，**在一个自定义的directive中使用一个自定义的filter**。刚开始我没有意识到这里面会涉及**依赖注入**的问题（唉，人蠢没办法啊！），导致走了不少弯路。

下面是具体的代码。

先是`html`部分的代码，

```html
<div ng-app="gxt" ng-controller="index">
    <!-- ... -->
    <site sites="sites" selected="selectedSite" change="changeSite(site)"></site>
    <!-- ... -->
</div>
```

然后是与之对应的`js`代码，

```javascript
var gxt = angular.module('gxt', []);

gxt.controller('index', ['$scope', function($scope) {
    $scope.sites = [{
        "id": "20112",
        "url": "http://www.baidu.com",
        "detected": true
    }, {
        "id": "20113",
        "url": "http://tieba.baidu.com",
        "detected": false
    }, {
        "id": "20114",
        "url": "http://ce.baidu.com",
        "detected": false
    }];

    $scope.changeSite = function(site) {
        console.log(site);
        // TODO do something with change callback
    };

    $scope.$watch('selectedSite', function(nv, ov) {
        console.log(nv, ov);
        // TODO check the model value
    });

}]);
```

显然，我这里自定义了一个directive，名字叫做`site`，其具体的实现如下，

```javascript
var directives = angular.module('directives', []);

directives.directive('sites', ['$filter', function ($filter) {
    return {
        restrict: 'EA',
        replace: true,
        scope: {
            sites: '=',
            selected: '=',
            change: '&'
        },
        template: [
            '<div class="site">',
                '<input type="text" class="site-input" ng-model="site" ng-change="manul()" ng-click="show($event)">',
                '<label class="placeholder" ng-show="showPlaceholder">输入网站url，如：www.baidu.com</label>',
                '<span class="arrow"></span>',
                '<ul class="" ng-show="toggleList">',
                    '<li ng-repeat="site in sites" ng-click="selectSite(site)">[[site.url | pureURL]]</li>',
                '</ul>',
            '</div>'
        ].join(''),
        link: function (scope, element, attrs) {
            scope.toggleList = false;
            scope.showPlaceholder = true;

            scope.show = function(ev) {
                scope.toggleList = true;

                ev.cancelBubble = true;
                ev.preventDefault();
                ev.stopPropagation();
            };

            $(document).on('click', function() {
                scope.$apply(function () {
                    scope.toggleList = false;
                });
            });
            scope.manul = function() {
                scope.showPlaceholder = scope.site ? false : true;
            };
            scope.selectSite = function(site) {
                scope.selected = site;
                scope.site = $filter('pureURL')(site.url);
                // 调用change回调
                //scope.change(site);
                scope.change({
                    site: site
                });
            };
        }
    };
}]);

```

从代码可以看出，这个`site`的作用是：**接受一个数组数据，然后渲染成一个下拉列表样式的可选项**，类似下图，

![](http://7xkwt1.com1.z0.glb.clouddn.com/如何在自定义的directive中使用自定义的filter-001.png)

注意，刚开始这里出现了问题，报错信息是`site.url | pureURl undefined is not a function`。

我很纳闷，看报错信息的意思应该是`template`中用到的`pureURL`未定义，但是我在之前的确是定义的了啊，其定义如下，

```javascript
var filters = angular.module('filters', []);

filters.filter('pureURL', [function () {
    var exports = function (input) {
        if (!input) {
            return '网址不能为空';
        } else {
            return input.replace(/(^(http|https):\/\/)|(\/$)/g, '');
        }
    };

    return exports;
}]);
```

在求助于google及stackoverflow之后，我猛然意识到，**我的directive module中没有注入filters**，这样在`site`中当然拿不到`pureURL`了啊。

所以，将`directives`的定义修改如下，

```javascript
var directives = angular.module('directives', ['filters']);
```

然后再修改`gxt`的声明如下，

```javascript
var gxt = angular.module('gxt', ['directives', 'filters']);
```

其实，这里也可以写成如下的形式，

```javascript
var gxt = angular.module('gxt', ['directives']);
```

即此可以省略`filters`的注入。因为在`directives`中已经注入了`filters`。


