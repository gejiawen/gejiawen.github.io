title: 'AngularJS:ngClass用法小解'
date: 2014-07-14 16:00:03
categories: [AngularJS]
tags: [ng, 转载]
---

`ngClass`是 **AngularJS** 预设的一个指令，用于动态自定义dom元素的css类。这里是官网给出的使用指南[AngularJS:ngClass](http://docs.angularjs.org/api/ng.directive:ngClass)

`ngClass`在实际的应用场景中还是比较灵活的，而在 **AngularJS** 中一般会有三种方式给元素的css class属性做一些门道。


# scope变量绑定（不推荐使用）


```html
<div class="{{className}}"></div>
```

```javascript
function ctrl($scope) {
    $scope.className = 'test-className';
}
```

**说明**
这种方式完全没有错，是AngularJS提供的一种改变class的方式，但是这种方式在`controller`中涉及了classname的赋值。在我看来似乎有点诡异。我希望的是`controller`是一个干净的纯净的`javascript`意义上的`object`。


# 字符串数组形式


```html
<div ng-class="{true: 'active', false: 'inactive'}[isActive]"></div>
```

```javascript
function ctrl($scope) {
    $scope.isActive = true;
}
```

**说明**
其结果可能会是2种情况，`isActive`表达式为`true`则会给`<div></div>`附加`active`样式，否则会附加`inactive`样式。


# 对象key/value处理


```html
<div ng-class {'selected': isSelected, 'car': isCar}"></div>
```

```javascript
function ctrl($scope) {
    $scope.selected = true;
    $scope.isCar = false;
}
```

**说明**
当`isSelect`为真值时增加`slected`样式，若`isCar`为真值，则再增加`car`样式。


# 参考列表

- [细说Angular ng-class](http://www.cnblogs.com/whitewolf/archive/2013/05/22/3092184.html)


End! All rights reserved `@gejiawen`.
