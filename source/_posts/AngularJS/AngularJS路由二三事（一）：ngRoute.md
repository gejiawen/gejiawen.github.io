postid: 'routes-solution-on-angular-ng-route'
title: AngularJS路由二三事（一）：ngRoute
categories: [AngularJS]
tags: [ng, AngularJS路由二三事]
date: 2015-12-15 10:41:43

---

**AngularJS路由二三事**专题文章，

- [AngularJS路由二三事（一）：ngRoute](http://blog.gejiawen.com/2015/12/15/routes-solution-on-angular-ng-route/)
- AngularJS路由二三事（二）：ui-router
- AngularJS路由二三事（三）：ui-router-extras

# 前言

在[AngularJS](https://angularjs.org/)中，关于路由的设计和用法是一个很重要的方面。简单来说AngularJS的路由其实是一种纯前端的解决方案。不同于后端路由，它的本质其实是：当请求一个url时，根据路由配置匹配这个url，然后请求模板片段，并插入到`ng-view`中去。所以从某种意义上来说，AngularJS的路由更加倾向通过改变url来进行页面的局部刷新。

**AngularJS路由二三事**这个专题文章中，我将基于AngularJS 1.5版本，结合内置的[ngRoute](https://docs.angularjs.org/api/ngRoute)服务、[ui-router](https://github.com/angular-ui/ui-router)模块，以及[ui-router-extras](https://github.com/christopherthielen/ui-router-extras)模块来详细阐述AngularJS中路由的相关内容。

文章中涉及到的示例代码在我的github上，就是这个[仓库](https://github.com/gejiawen/angular-route-demo)，可供参考。

另外提一点，因为AngularJS的官网在国内访问可能不太稳定，所以可能对查阅文档造成一些干扰。我们可以选择查阅[AngularJS中文站](http://angularjs.cn/)提供的[文档镜像](http://docs.angularjs.cn/api)，但是这个文档并不是紧跟AngularJS官方的版本号的。另一种途径就是，我们可以将AngularJS的[源代码](https://github.com/angular/angular.js)clone到本地，然后安装好所有的依赖之后在本地build一下，然后`grunt webserver`就可以在本地起一个AngularJS的官方网站，此时就可以无障碍的查阅相关文档了。

# `ngRoute`

本篇文章我们将介绍如何使用AngularJS内置的`ngRoute`模块来做前端路由。

我不太记得AngularJS是从哪个版本开始将`ngRoute`独立成一个单独的module，貌似是1.2之后吧，现在如果要使用`ngRoute`需要额外加载这个模块文件，就像下面这样，

```html
<script src="../bower_components/angular/angular.js"></script>
<script src="../bower_components/angular-route/angular-route.js"></script>
```

除了**angular-route**模块，还有**angular-animate**，**anglar-aria**，**angular-cookies**等模块在使用时也需要额外引入相关文件。这地方有点小坑，大家注意一下就可以了。

## 使用说明

`ngRoute`模块中包含以下内容，

| 名称 | 所属 | 作用 |
| :--- | :--- | :--- |
| `ngView` | DIRECTIVE | 提供不同路由模板插入的视图层 |
| `$routeProvider` | PROVIDER | 提供路由配置 |
| `$route` | SERVICE | 用于构建各个路由的url、view、controller这三者的关系 |
| `$routeParams` | SERVICE | 解析返回路由中带有的参数 |

上表中的每一个组件在路由中都扮演着不可或缺的作用。基本上使用AngularJS配置路由的基本流程是这样的，

1. 在主模板中使用`ngView`定义一个路由模板的视图层。不同路由对应的模板将会插入到这个`ngView`所在的dom元素下。
2. 使用`$routeProvider`进行路由配置，包括每一个路由对应的url，template以及controller。除了这些基本的配置之外，还会有一些额外的配置，比如`resolve`配置等。
3. 在每个路由的controller中完成对应的业务逻辑。
4. 可以通过注入`$routeParams`服务来获取路由url上的参数；还可以通过`$rootScope`来监控`$routeChangeStart`和`$routeChangeSuccess`事件。


## 简单实例

在实例代码[仓库](https://github.com/gejiawen/angular-route-demo)中有一个**demo001**文件夹，其目录结构如下，

```javascript
- index.html
- home.html
- post.html
- about.html
- index.js
```

其中`index.html`是我们的主页面文件，其内容如下，

```html
<body ng-app="demo001" ng-controller="Demo">
    <h1>Angular Route Demo</h1>
    <hr>
    <div>
        <a href="#/home">home</a>
        <a href="#/post">post</a>
        <a href="#/about">about</a>
    </div>
    <hr>
    <div ng-view></div>
</body>
```

注意，我们的页面上有一个`ng-view`指令。

在`index.js`中，我们需要声明一个AngularJS的module叫做*demo001*，并且做一些路由配置工作。代码如下，

```javascript
angular.module('demo001', ['ngRoute'])
.config([
    '$routeProvider',
    function ($routeProvider) {
        $routeProvider
            .when('/home', {
                templateUrl: 'home.html',
                controller: 'HomeController'
            })
            .when('/post', {
                templateUrl: 'post.html',
                controller: 'PostController'
            })
            .when('/about', {
                templateUrl: 'about.html',
                controller: 'AboutController'
            })
            .otherwise('/home')
    }
])
```

这里有3点需要注意，

1. 在声明`demo001`这个module的时候，需要将`ngRoute`作为依赖。否则报`$routeProvider`未定义这样的错误。
2. 在module的configuration中，我们调用`$routeProvider.when`来配置不同路由的具体信息。`$routeProvider.when`方法接受2个参数，第一个是路由的url。第二个路由的具体配置，包括对应的模板地址，控制器名称等。
3. `$routeProvider.otherwise`可以用于设置默认路由地址。简单来说就是将未设置的url自动重定向到此url。

在我们补充完各个路由的控制器后，我们打开`index.html`就可以预览了。在预览时，注意点击不同链接时url的变化，还可以观察浏览器的Network行为。所有的子模板默认加载一次之后就会被缓存。

## 模块化实例

在经过上面的实例之后，应该对AngularJS路由的基本用法有所了解了。现在我们来假定有这样一个场景，假设我们的项目比较复杂，内部的模块很多。此时更优的一种方案是，基于AngularJS来做模块化设计与开发。AngularJS的模块化是以它的module以及依赖注入等行为作为基础的。

在实例代码[仓库](https://github.com/gejiawen/angular-route-demo)中有一个**demo002**的文件夹，其目录结构如下，

```javascript
- index.html
- index.js
- home.html
- home.js
- post.html
- post-id.html
- post.js
- about.html
- about.js
```

`index.html`与上一个实例相比基本没有变化。然后我们再看一眼`index.js`，

```javascript
angular.module('demo002', [
    'ngRoute',
    'Module.Home',
    'Module.Post',
    'Module.About'
])

.config([
    '$routeProvider',
    function ($routeProvider) {
        $routeProvider.otherwise('/home');
    }
])
```

与之前不同的是，我们在声明`demo002`这个module时，附加了额外3个module。在路由的配置中，也仅仅只有一个`$routeProvider.otherwise`的设置。

这里我们就是使用了模块化的思想，将`/home`，`/post`，`/about`这几个路由抽象成独立的module，将他们内部的所有逻辑和设置都封装在内部。比如下面的`home.js`

```javascript
angular.module('Module.Home', ['ngRoute'])

.config([
    '$routeProvider',
    function ($routeProvider) {
        $routeProvider.when('/home', {
            templateUrl: 'home.html',
            controller: 'HomeController'
        });
    }
])

.controller('HomeController', [
    '$scope',
    function ($scope) {
        $scope.msg = 'This is home page';
    }
]);
```

AngularJS的module机制和依赖注入机制，为模块化设计提供了基础。在稍微复杂一点的angularjs项目中我是非常推荐使用模块化开发的，能够抽象成独立module的不仅仅是不同的路由模块，可以是一个公共组件，也可以是一个公共服务等等。

## 路由参数

在上一个**模块化实例**中，我们的`post.js`模块如下，

```javascript
angular.module('Module.Post', ['ngRoute'])

.config([
    '$routeProvider',
    function ($routerProvider) {
        $routerProvider
            .when('/post', {
                templateUrl: 'post.html',
                controller: 'PostController'
            })
            .when('/post/:postId', {
                templateUrl: 'post-id.html',
                controller: 'PostIdController'
            })
    }
])

.controller('PostController', [
    '$scope',
    function ($scope) {
        $scope.posts = [
            {
                name: 'post1',
                id: 'post-001'
            }, {
                name: 'post2',
                id: 'post-002'
            }
        ]
    }
])

.controller('PostIdController', [
    '$scope',
    '$routeParams',
    function ($scope, $routeParams) {
        $scope.msg = 'post id: ' + $routeParams.postId;
    }
]);
```

注意这里，我们使用`$routeProvider`配置的第二个路由是这样的`/post/:postId`。路由中的`/:postId`其实是一个参数，它将匹配类似`/post/001`这种url，其中**001**就是这个`:postId`的值。

我们在路由对应的控制器中，可以通过`$routeParams`参数来获取这个url参数。如下，

```javascript
.controller('PostIdController', [
    '$scope',
    '$routeParams',
    function ($scope, $routeParams) {
        $scope.msg = 'post id: ' + $routeParams.postId;
    }
]);
```

依次类推，我们可以为路由的url设置多个参数，比如`/post/:year/:month/:day/:postName`这样的路由，它将匹配*/post/2015/12/15/angular-router-demo*这样的路径。控制器中注入的`$routeParams`服务将会得到类似下面的对象，

```javascript
{
    "year": 2015,
    "month": 12,
    "day": 15,
    "postName": "angular-router-demo"
}
```

## 路由中的resolve

在前面我们已经说明，可以使用`$routeProvider.when`方法进行路由配置。这个`$routeProvider.when`方法接受2个参数，其中第一个是路由的url，第二个是路由的具体配置项目。

关于[$routeProvider.when](https://docs.angularjs.org/api/ngRoute/provider/$routeProvider#when)的具体用法可以参考官方的文档。

这里我仅仅针对其中的一个配置项`resolve`进行一些说明。

我们先来假设一个场景。

比如我最近上班太累了，想来一场旅行。在旅行之前，我需要拿到一张机票。而旅游网站出票是需要时间的。

将这个场景抽象成AngularJS应用就是这样的：

1. 有两个页面，一个是上班页面（`/home`），一个是拿到机票开始旅行页面（`/trip`）。
2. 默认处于上班页面。可以通过导航到开始旅行页面。
3. 在进入旅行页面之前，我们必须要有一张机票。

所以，这个场景中，我们的问题可以总结成，当我从`/home`进入`/trip`路由之前，必须要拿到一个机票数据。

在实例代码[仓库](https://github.com/gejiawen/angular-route-demo)中有一个**demo003**文件夹，其目录结构如下，

```javascript
- index.html
- index.js
- index2.js // resolve方案
- home.html
- trip.html
```

在`index.js`中，

```javascript
.config([
    '$routeProvider',
    function ($routeProvider) {
        $routeProvider
            .when('/home', {
                templateUrl: 'home.html',
                controller: 'HomeController'
            })
            .when('/trip', {
                templateUrl: 'trip.html',
                controller: 'TripController'
            })
            .otherwise('/home');
    }
])

.controller('TripController', [
    '$scope',
    '$timeout',
    function ($scope, $timeout) {
        $timeout(function () {
            $scope.ticket = '上海 -> 澳大利亚'
        }, 4000);
    }
])
```

这里我们使用定时器`$timeout`来模拟一个耗时的出票操作。此时我们从`/home`->`/trip`时，页面会白屏4秒钟。意味着在进行url跳转完毕的时候，我们就已经将`/trip`的模板插入到了`ng-view`中，但是此时`/trip`需要的数据还没有准备好。

这种场景下，我们一般会有两种方式去解决这个问题，

1. 第一种方式：在`/trip`的模板和控制器中做一些视觉等待逻辑。比如在`TripController`中进行耗时操作时，我们可以临时展示一个loading视觉，待耗时操作完毕之后，我们再将这个视觉隐藏即可。
2. 第二种方式：在配置路由时，配置`resolve`选项。配置`resolve`选项意味着，在进入这个路由之前就必须等待`resolve`中的数据返回。

这里我们主要来看看第二种方式。在实例代码仓库的**demo003**文件夹下的`index2.js`中，我们是怎么做的，

```javascript
.config([
    '$routeProvider',
    function ($routeProvider) {
        $routeProvider
            .when('/home', {
                templateUrl: 'home.html',
                controller: 'HomeController'
            })
            .when('/trip', {
                templateUrl: 'trip.html',
                controller: 'TripController',
                resolve: {
                    ticket: ['$q', '$timeout', function ($q, $timeout) {
                        var deferred = $q.defer();
                        $timeout(function () {
                            deferred.resolve('上海 -> 澳大利亚');
                        }, 4000);
                        return deferred.promise;
                    }]
                }
            })
            .otherwise('/home');
    }
])

.controller('TripController', [
    '$scope',
    'ticket',
    function ($scope, ticket) {
        $scope.ticket = ticket;
    }
])
```

注意`/trip`路由的配置中，我们设置了一个`resolve`配置项。这个配置项包含了一个叫做`ticket`的key，它将返回一个promise（这里采用AngularJS内置的[$q](https://docs.angularjs.org/api/ng/service/$q)来实现promise）。其内部也是使用定时器做了一个耗时操作的模拟。

当我们的路由从`/home`->`/trip`时，会触发`resolve`下的所有promise，只有当所有的promise都都被正确的resolve之后才会进行路由切换，才会将`/trip`的模板插入到`ng-view`中。其实此时`$route`会抛出一个`$routeChangeSuccess`的事件，这个事件会被`$rootScope`捕获到。

若`resolve`中只要有一个promise没有被正确的resolve，那么此时`$route`将会抛出一个`$routeChangeError`的事件，并且终止路由切换，虽然url中的地址可能的确发生了变化，但是`/trip`的模板并没有插入到`ng-view`，且`TripController`也没有被执行。

当所有的`resolve`配置都返回之后，AngularJS会将`resolve`中key作为对应控制器的一个依赖注入进去，然后我们在相应的controller中就可以使用了。比如，

```javascript
.controller('TripController', [
    '$scope',
    'ticket',
    function ($scope, ticket) {
        $scope.ticket = ticket;
    }
])
```

这里可以看出，上面提到的两种**预载入数据**的方案其实是有着本质区别的。前者其实是在跳转的目标路由上做一些额外的工作去适配耗时操作的视觉，此时目标路由的模板已经被载入`ng-view`，且相应的控制器也被执行了。而后者在跳转目标路由之前做一些额外工作去预加载数据，当数据准备妥当才会去载入目标路由的模板和执行相应的controller。

关于在路由中使用`resolve`的示例，有兴趣可参阅[这篇文章](http://www.undefinednull.com/2014/02/17/resolve-in-angularjs-routes-explained-as-story/)以及[这篇文章](http://odetocode.com/blogs/scott/archive/2014/05/20/using-resolve-in-angularjs-routes.aspx)。












