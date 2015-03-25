title: AngularJS的ngTransclude
date: 2014-07-17 15:51:45
categories: [AngularJS]
tags: [ng]
---

# transclued的定义

**Transclude**好像并不是一个英语单词，有道词典没有，百度翻译的意思是*嵌入*的意思。`transclude`在angularjs的自定义directive中是比较常见的一个东西，所以非常有必要了解一下。

我们首先看下官方api对`ng-transclude`的解释：

> Directive that marks the insertion point for the transcluded DOM of the nearest parent directive that uses transclusion. Any existing content of the element that this directive is placed on will be removed before the transcluded content is inserted.

# 官方示例及解释

官方例子：

```html
<div ng-controller="ctrl">
    <input ng-model="title"></br>
    <textarea ng-model="text"</textarea></br>
    <pane title="{{title}}"</pane>
</div>
```

这里的`pane`是一个自定义directive，标签里还有一个表达式，这个指令的目的是显示input中输入title，和textarea中输入的text，当然是按照一定的dom结构显示。看下pane是如何实现：

```javascript
app.directive('pane', function() {
    return {
      restrict: 'E',
      transclude: true,
      scope: {
          title:'@'
      },
      template:
            '<div style="border: 1px solid black;">' +
                '<div style="background-color: gray">{{title}}</div>' +
                '<div ng-transclude></div>' +
            '</div>'
    };
});
```

首先是我们想把`<pane title="{{title}}">{{text}}</pane>`中`{{title}}`和`{{text}}`变量的内容封装到我们的dom结构中，`{{title}}`可以通过结构`scope: { title:'@' }`取得，但是我们想要保留`<pane></pane>`标签里的东西（有可能会是很多的表达式和dom结构），那就需要今天的主角`transclude`了。

这个例子的结果生成的dom结构是这样的：

```html
<div style="border: 1px solid black;">
    <div style="background-color: gray">我是标题</div>
    我是内容
</div>
```

有个这样结果我想你就看明白了。**原来模板中的`<div ng-transclude></div>`最后会被`<pane></pane>`标签里的表达式内容所替换。这是就是transclude的用途。**

我们回过头再来看ng-transclude的定义：
* ng-transclude指明的是一个插入的位置
* 指令中标签里的元素都会先删除然后被嵌入包含后的内容所替换


这个例子够简单，这也是最基础的用法，我们再来看下高级一点的用法。我们对上面的例子进行扩充，加上了类型和时间:

```html
<div ng-controller="Ctrl">
    <input ng-model="title"><br>
    <input ng-model="type"><br>
    <input ng-model="time"><br>
    <textarea ng-model="text"></textarea> <br/>
    <pane title="{{title}}">
        <span class="time">time</p>
        <p class="type">{{type}}<p>
        <p class="content">{{text}}<p>
    </pane>
</div>
```

最终的目的是这样的：

```html
<div style="border: 1px solid black;">
    <div style="background-color: gray">
        我是标题<span class="time">我是时间</span>
    </div>
    <p class="type">我是分类</p>
    <p class="content">我是内容</p>
</div>
```

光一个ng-transclude是不行的，当然你也可以像title那样传参，但现在是在学习transclude，没有transclude还学个毛啊。我们有两种方法可以实现这个目的。


# 使用compile函数的transclude参数

先看pane的directive代码：

```javascript
app.directive('pane', function() {
    return {
        restrict: 'EA',
        template:
            '<div style="border: 1px solid black;">' +
                '<div class="title" style="background-olor: gray">{{title}}</div>' +
            '</div>',
        replace: true,
        transclude: true,
        compile: function(element, attrs, transcludeFn) {
            return function (scope, element, attrs) {
                transcludeFn(scope, function(clone) {
                    var title= element.find('title');
                    var time = clone.find('.time');
                    var type = clone.find('.type');
                    var text= clone.find('.content');

                    title.append(time);
                    element.append(type);
                    element.append(text)
                });
            };
        }
    };
});
```

`transcludeFn`是一个function：`transcludeFn(scope, function(clone){})`作用域和嵌入包含的内容，clone嵌入的内容是被jquery封装过的，有了它，我们就可以做任何想要做的dom操作了。

# 在controller里注入$transclude

先上代码：

```javascript
app.directive('pane', function() {
    return {
        restrict: 'EA',
        template:
            '<div style="border: 1px solid black;">' +
                '<div class="title" style="background-olor: gray">{{title}}</div>' +
            '</div>',
        replace: true,
        transclude: true,
        controller: [
            '$scope', '$element', '$transclude',
            function ($scope, $element, $transclude) {
                $transclude(function(clone, scope) {
                    var title= element.find('title');
                    var time = clone.find('.time');
                    var type = clone.find('.type');
                    var text= clone.find('.content');

                    title.append(time);
                    element.append(type);
                    element.append(text)
                });
            }
        ]
    };
});
```

换汤不换药，其实就是`$transclude`，`transcludeFn`这两个函数执行的地方不同。里面是一模一样的。

还有一个需要说明的是transclude的作用域的问题。

在官方文档中提到过deretive的作用域是单独的，`transclude`也创建了一个单独的作用域，而且与derectvie的作用域是平行的，还是拿上面的例子来说。

```html
<div ng-controller="Ctrl">...</div>
```

首先controller Ctrl会创建一个作用域scope1，derective Pane会在scope1下面创建一个scope2，scope1 包含 scope2，tranclude又会在scope1下面创建一个scope3，scope1也包含scope3，scope2和scope3是兄弟关系，平行的两个子作用域。

好，我们可以来验证一下：

在controller中加日志，

```javascript
function Ctrl($scope) {
    $scope.title = 'Lorem Ipsum';
    $scope.text = 'Neque porro quisquam est qui dolorem ipsum quia dolor...';
    console.log('scope1', $scope);
}
```

在derective中添加日志，

```javascript
app.directive('pane', function() {
    return {
        restrict: 'EA',
        template:
            '<div style="border: 1px solid black;">' +
                '<div class="title" style="background-olor: gray">{{title}}</div>' +
            '</div>',
        replace: true,
        transclude: true,
        controller: [
            '$scope', '$element', '$transclude',
            function ($scope, $element, $transclude) {
                console.log('scope2', $scope)
                $transclude(function(clone, scope) {
                    console.log('scope3', scope);
                    var title= element.find('title');
                    var time = clone.find('.time');
                    var type = clone.find('.type');
                    var text= clone.find('.content');
                    title.append(time);
                    element.append(type);
                    element.append(text)
                });
            }
        ],
    };
});
```

在控制台可以看到，

```shell
scope1 a {$id: "003", this: a, $$listeners: Object, $parent: e, $$childTail: null…}
scope2 e {$id: "004", $$childTail: null, $$childHead: null, $$prevSibling: null, $$nextSibling: null…}
scope3 a {$id: "005", this: a, $$listeners: Object, $parent: a, $$childTail: null…}
```

点开`scope2`和`scope3`，就能看到`$paren`t的Id为003，这就印证了我们的观点。




End! All rights reserved `@gejiawen`.
