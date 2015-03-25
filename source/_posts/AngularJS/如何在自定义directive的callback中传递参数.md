title: "如何在自定义directive的callback中传递参数"
date: 2014-10-24 16:48:16
categories: [AngularJS]
tags: [ng, 实践]
---

**directive**是angularjs中一个比较复杂的概念，是用户可自定义html标签的重要手段，本博客之前有[一篇文章](http://gejiawen.github.io/2014/07/16/AngularJS/AngularJS-Directive%E7%94%A8%E6%B3%95%E8%AF%B4%E6%98%8E/)对angularjs中directive的用法做了非常详细的说明。

这篇文章[有个部分](http://gejiawen.github.io/2014/07/16/AngularJS/AngularJS-Directive%E7%94%A8%E6%B3%95%E8%AF%B4%E6%98%8E/#关于scope)提到了directive中关于`scope`的定义说明。

还是[上篇文章](http://gejiawen.github.io/2014/10/24/AngularJS/%E5%A6%82%E4%BD%95%E5%9C%A8%E8%87%AA%E5%AE%9A%E4%B9%89%E7%9A%84directive%E4%B8%AD%E4%BD%BF%E7%94%A8%E8%87%AA%E5%AE%9A%E4%B9%89%E7%9A%84filter/)的那个例子，

```javascript
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

请注意这里，

```javascript
scope: {
    sites: '=',
    selected: '=',
    change: '&'
}
```

其中，`scope.change`的值为`&`，从前面的文章中，我们可以知道，他表示这个directive在html中会有一个名字为`change`的attribute。而且这个attribute的类型还必须是一个函数。

html和js的代码如下，

```html
    <!-- ... -->
    <site sites="sites" selected="selectedSite" change="changeSite(site)"></site>
    <!-- ... -->
```

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

值得注意的地方是，js的controller中，方法`$scope.changeSite`是带有一个默认参数的，而这个参数又是从directive的定义中传递过来的，如下，

```javascript
// ...

scope.selectSite = function(site) {
    scope.selected = site;
    scope.site = $filter('pureURL')(site.url);
    // 调用change回调
    //scope.change(site);
    scope.change({
        site: site
    });
};
```

这里我们在执行change回调的时候，采用`{site: site}`这种形式，将参数传递给js controller中对应的方法。

下面是之前文章的解释，

> 一般来说，我们希望通过一个表达式，将数据从isolate scope传到parent scope中。这可以通过传送一个本地变量键值的映射到表达式的wrapper函数中来完成。例如，如果表达式是`increment(amount)`，那么我们可以通过`localFn({amount:22})`的方式调用localFn以指定amount的值。



End! All rights reserved `@gejiawen`.
