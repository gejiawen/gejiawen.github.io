title: 'AngularJS:select指令用法详解'
date: 2014-07-14 14:49:28
categories: [AngularJS]
tags: [ng]
---

`select`指令是 **AngularJS** 预设的一组`directive`。这里是AngularJS官网给出的用法： [AngularJS:select](http://docs.angularjs.org/api/ng.directive:select)

大概的意思是，`select`中的`ngOption`可以采用和`ngRepeat`指令类似的循环结构，其数据源可以是`Array`或者是`Object`。针对两种数据源又可以衍生出好几种用法。

但是官网给出的例子太少了。

下面是针对几个不太容易理解的用法示例。


# 先上Controller

```javascript
function selectCtrl($scope) {
    $scope.selected = '';

    $scope.model = [{
        id: 10001,
        mainCategory: '男',
        productName: '水洗T恤',
        productColor: '白'
    }, {
        id: 10002,
        mainCategory: '女',
        productName: '圆领短袖',
        productColor: '黑'
    }, {
        id: 10003,
        mainCategory: '女',
        productName: '短袖短袖',
        productColor: '黄'
    }];
}
```

# 示例

## 示例一：基本下拉效果


**usage:**
`label for value in array`

```html
<select ng-model="selected" ng-options="m.productName for m in model">
    <option value="">-- 请选择 --</option>
</select>
```

**效果**
![基本下拉效果](http://7xkwt1.com1.z0.glb.clouddn.com/AngularJS-select指令用法详解-001.png)

**说明**
1. **usage** 中的`value`也就是`ng-options`中的`m`，而`m`是数组`model`的一个元素，*它是一个变量*
2. **usage** 中的`label`也就是`ng-options`中的`m.productName`，其实就是一个[AngularJS Expression](http://docs.angularjs.org/guide/expression)


## 示例二：自定义下拉显示名称

**usage**
`label for value in array`

```html
<select ng-model="selected" ng-options="(m.productColor + ' - ' + m.productName) for m in model">
    <option value="">-- 请选择 --</option>
</select>
```

**效果**
![自定义下拉显示名称](http://7xkwt1.com1.z0.glb.clouddn.com/AngularJS-select指令用法详解-002.png)

**说明**
1. 可以看到，**usage** 中的`label`可以根据需求拼接出不同的字符串


## 示例三：让选项分组

**usage**
`label group by groupName for value in array`

```html
<select ng-model="selected" ng-options="(m.productColor + ' - ' + m.productName) group by m.mainCategory for m in model">
    <option value="">-- 请选择 --</option>
</select>
```

**效果**
![让选项分组](http://7xkwt1.com1.z0.glb.clouddn.com/AngularJS-select指令用法详解-003.png)

**说明**
1. 这里使用了`group by`，通过`$scope.model`中的`mainCategory`字段进行分组


## 示例四：自定义`ngModel`的绑定

**usage**
`select as label for value in array`

```html
<select ng-model="selected" ng-options="m.id as m.productName for m in model">
    <option value="">-- 请选择 --</option>
</select>
```

**效果**
![自定义ngModel的绑定](http://7xkwt1.com1.z0.glb.clouddn.com/AngularJS-select指令用法详解-004.png)

**说明**
1. 这种用法也是`select`指令中较为复杂的一种。从效果中可以看出，**usage** 中`select`的作用就是重新绑定`ng-model`的值。在这里，`ng-model`等于`m.id`，当`ng-model`发生改变的时候，得到的实际值是`m.id`的值。


End! All rights reserved `@gejiawen`.


