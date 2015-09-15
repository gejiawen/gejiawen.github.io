postid: "different-from-angularjs-directive-controller-service"
title: AngularJS中使用Directive、Controller、Service
date: 2014-07-14 16:52:33
categories: [AngularJS]
tags: [ng]
---

写在前面：本文适合AngularJS新手，老手略过。

 AngularJS是一款非常强大的前端MVC框架。同时，它也引入了相当多的概念，这些概念我们可能不是太熟悉。

 * Directive 指令
 * Controller 控制器
 * Service 服务

下面我们逐个来看这些概念，研究一下为什么它们会像当初设计的那样强大，同时研究一下为什么我们要以那样的方式去使用它们。

本文系转载，[点我看原文](http://damoqiongqiu.iteye.com/blog/1971204)，版权归原作者所有。


# SERVICES 服务

如果你已经使用过AngularJS，你可能已经遇到过`Service`这个概念了，简而言之，`Service`就是 **单例对象** 在AngularJS中的一个别名。这些小东西（指单例对象）会被经常传来传去，保证你每次访问到的都是同一个实例。这一点和工厂模式不同。

基于这种思想，单例对象让我们可以实现一些相当酷的功能，它可以让很多`controller`和`directive`访问内部的数值。

我们来看一个非常常见的问题，那就是在应用中的不同代码块之间如何共享数据？

我们首先创建一个module（模块），本文中的所有代码都会用到这个module。

```javascript
// 创建一个angularjs的module
var module = angular.module('my.new.module', []); // 这里的第二个空数组参数一定要有
```

下一步，我们来创建一个新的service（服务）。假设我们上面的这个module是用来管理图书的。所以，这里我们来创建一个Book Service，然后把一个JSON对象数组添加到这个service中。

```javascript
module.service('Book', [
    '$rootScope',
    function($rootScope) {
        var service = {
            books: [{
                title: 'Magician',
                author: 'Raymond E, Feist'
            }, {
                title: 'The Hobbit',
                author: 'J.R.R Tolkien'
            }],
            addBook: function(book) {
                service.books.push(book);
                $rootScope.$broadcast('books.update');
            }
        };

        return service;
    }
]);
```

这个一个非常简单的service（有时候这样的service其实就够用了）。我们这里正在做的事情就是在管理一个book数组，同时还带有一个`addBook`方法，在有需要的时候可以添加更多的书籍。`addBook`方法还会在application上广播一个事件，告诉所有正在我们服务`Book`的人，数组已经被更新了，从而让它们自己也做一些刷新操纵。

现在，我们要做的就是把这个service传递给各种controller、directive、filter，或者其他任何需要它的东西，然后它们就可以访问service中提供的这些方法和属性了。

下面我们声明一个controller：

```javascript
module.controller('books.list', [
    '$scope',
    'Book',
    function($socpe, Book) {
        // books.update事件监听
        $scope.$on('books.update', function(event) {
            $scope.books = Book.books;
            $scope.$apply();
        });

        $scope.books = Book.books;
    }
]);
```

同样非常简单。我们上面所做的就是为了我们的`module`创建了一个新的`controller`。在创建的时候把`$scope.provider`和我们自己的`Book service`传递给了它。能明白我们在干嘛吗？我们把前面创建的`Book service`中的`books`数组赋给了`controller`内部的局部$scope对象。很酷，对吧？

 好，这里的核心问题是什么呢？我们节省了一些时间，并且在`controller`上创建了一个数组。对，我们确实这样做了。这样做确实也为我们节约了一点时间，但是如果我们要**在其它地方处理这些书籍信息**应该怎么办呢？通过`$scope`来维护数据是非常粗暴的一种方式。由于其它`controller`、`directive`、`model`的影响，`$scope`很容易就会崩溃或者变脏。它很快就会变成一团乱麻。通过一种集中的途径（在这里 就是service）来管理所有书籍数据，然后通过某种方式来请求修改它，这样不仅仅**会更加清晰**，同时当应用的体积不断增大的时候也**更加容易管理**。最后，它还可以让你的**代码保持模块化**（这也是Angular很擅长的一件事情）。一旦你在其它项目中需要用到这个`service`，你没有必要在`$scope`、 `controller`、`filter`等等东西里面到处去查找相关的代码，因为**所有东西都在`service`里面**！

好。那么我们什么时候应该使用service呢？答案是：**无论何时**，当我们需要在不同的域中共享数据的时候。另外，多亏了Angular的 **依赖注入系统** ，实现这一点是很容易并且很清晰的。


# CONTROLLERS 控制器

我们再来看控制器！

除非你曾经使用过前端MVC，否则从服务端MVC的思维模式转向客户端MVC的思维模式就如同一次脑筋急转弯。为什么会这样呢？

这是因为，虽然在前端开发中`controller`实现了非常类似的功能，但是它同时还会实现一些与服务端controller非常不同的功能。在 Angular中，`controller`自身并不会处理**request**，除非它是用来处理路由(route)的（很多人把这种方式叫做创建`route controller`，路由控制器），更明确地说，尤其是你的应用里面那些作为界面的一部分的controller，它们只会管理非常小的一段代码。

controller应该纯粹地用来把service、依赖关系、以及其它对象串联到一起，然后通过$scope把它们关联到view上。如果在你的视图里面需要处理复杂的业务逻辑，那么把它们放到controller里面也是一个非常不错的选择。回到我们前面的这个books例子，我实际上并没有什么东西需要添加到controller里面。

但是如果我要add一本书籍应该怎么办呢？我应该在controller上面新增一个方法来处理这件事情吗？ 不，原因在下面解释。因为它是DOM交互/操作的一部分。所以请把它放到directive(指令)里面。怎么做呢？很高兴你能问出这个问题。


# DIRECTIVES 指令

到目前为止，在我们所编写的大量AngularJS应用中，应用中最主要的复杂部分都在directive（指令）中。有一个强大的工具可以用来操作和修改DOM，它也是我们这里需要讨论的内容。我们来提供一个按钮，用户通过它可以向service里面添加一本图书，以这一功能来结束此文。

一个常见的反模式（即反面教材）是在controller里面添加DOM交互代码。Angular对`directive`的定义是一段代码片段，你可以用它来操作DOM，但是我觉得`directive`也是进行用户交互的很好选择。我们来扩展前面的例子，为用户提供一个按钮，通过这个按钮可以向`service`里面添加一本书籍。

```javascript
module.directive("addBookButton", [
    'Book',
    function(Book) {
        return {
            restrict: "A",
            link: function(scope, element, attrs) {
                element.bind( "click", function() {
                    Book.addBook({
                        title: "Star Wars",
                        author: "George Lucas"
                    });
                });
            }
        }；
    ｝
]);
```

很简单的东西。我们创建了一个指令，它的核心目的是简单地向books列表中添加一本书籍，books已经注册在了我们的Book服务中。我们来把这个指令应用到我们的视图中。

```html
<button add-book-button>Add book</button>
```

如你所见，我们仅仅把指令当作一个元素属性来使用。每次点击这个按钮的时候，它都会把《Star Wars》（《星球大战》）这本书添加到我们的`Book service`中去。简单、轻松、模块化，并且易复用。

好了，我们为什么不直接在控制器上面添加一个addBook之类的方法呢，比如说就像下面这样：

```javascript
$scope.addBook = function() {
    Book.addBook({
        title: "Star Wars",
        author: "George Lucas"
    });
};
```

这样我们也能获得同样的结果，对吧？是的，确实如此，但是这样做会带来一个重大的问题。

一旦我需要在其它地方添加书籍，我必须拷贝这份代码（非常un-DRY！，DRY---Dont Repeat Yourself，貌似是Ruby所倡导的一个重要的编码原则。），或者进行重构（重构本身并不是什么不好的的事情）。通过直接构建一个指令的方式，我们以后就没有必要担心这种事情了，同时下次再需要实现相同功能的时候完全不需要花任何时间。通过构建指令的方式来进行DOM交互和修改，随着业务需求的不断介入，我们就可以立即腾出手来处理复杂性不断增加的应用了。这是相当不错的一件事情，因为它保证了我们可以更少地和自己的实现打架，并且可以一直编写**DRYer code**。

Angular的模块依赖哲学无疑让它成为了一款非同凡响的框架。它让我们能够以这样一种方式来编写我们的前端代码：我们不会干翻自己，也不会干翻框架---这可能是它最强大的力量。

希望我已经充分说明了你应该在何时何地使用这几个Angular概念，从而能够更好地编写你自己的代码。

[英文原文](http://kirkbushell.me/when-to-use-directives-controllers-or-services-in-angular/)



