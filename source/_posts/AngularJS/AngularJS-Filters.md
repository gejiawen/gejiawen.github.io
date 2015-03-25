title: 'AngularJS Filters'
date: 2014-07-14 17:53:30
categories: [AngularJS]
tags: [ng]
---

本文将简单介绍AngularJS中的filter组件，即AngularJS的过滤器。

在正文之前，先来回顾一下ng中的一些基本概念：

* `module`，代码的组织单元，其它东西都是在具体的模块中定义的
* `app`，业务概念，可能会用到多个模块
* `service`，仅在数据层面实现特定业务功能的代码封装
* `controller`，与DOM结构相关联的东西，既是一种业务封装概念，又体现了项目组织的层级结构
* `filter`，改变输入数据的一种机制
* `directive`，与DOM结构相关联的、特定功能的封装形式

过滤器`Filter`的自定义是最简单的，就是一个函数，接受输入，然后返回结果。在考虑过滤器时，我觉得很重要的一点：**无状态**。


具体来说，过滤器就是一个函数，函数的本质含义就是确定的输入一定得到确定的输出。虽然`filter`是定义在`module`当中的，而且`filter`又是在`controller`的DOM范围内使用的，但是，它和具体的`module`，`controller`，`$scope`这些概念都没有关系（虽然在这里你可以使用js的闭包机制玩些花样），它仅仅是一个函数，而已。换句话说，它没有任何上下文关联的能力。

下面是一个简单点例子：

```javascript
var app = angular.module('Demo', [], angular.noop);

app.filter('map', function(){
    var filter = function(input){
      return input + '...';
    };

    return filter;
});
```

当有多个参数时，html中使用`:`来分割，形如：
`data|map2:province:city`


End! All rights reserved `@gejiawen`.
