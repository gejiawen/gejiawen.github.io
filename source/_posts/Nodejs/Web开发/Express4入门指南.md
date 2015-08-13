title: "Express4入门指南"
date: 2015-08-13 14:52:29
categories: [Nodejs]
tags: [nodejs, 漫游NodeJS系列]

---

[Express](http://expressjs.com/)是基于[Nodejs](https://nodejs.org/)以及一系列Nodejs第三方package的一款Web开发框架。

Express经历过2.x，3.x以及最新的4.x版本。Express各个版本的差异还是比较大的。现在2.x版本官方已经不再维护（deprecated），3.x版本虽然可以使用，但是官方建议升级到4.x版本。如果你之前使用的express 3.x版本，官方也有给出迁移到4.x的[指南](http://expressjs.com/guide/migrating-4.html)。

本文围绕express的Routing和Middleware机制，作一些讨论。主要参考express官网的guide。

# 安装和基本示例

我们首先从最基本的讲起。首先我们得有一个nodejs项目，

```bash
> mkdir test & cd test
> npm init
```

我一般习惯使用`npm init`来创建一个nodejs项目。因为它可以提供一个交互式的命令行来生成最基本的`package.json`文件。

在创建好`package.json`文件之后，接下来我们就需要安装express了，

```bash
> npm install express --save
```

我们使用`--save`选项自动更新`package.json`的`dependencies`字段。在express安装结束之后，`package.json`就变成了下面这样了，

```json
{
  "name": "test",
  "version": "1.0.0",
  "description": "kick off express4",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "keywords": [
    "nodejs"
  ],
  "author": "gejiawen",
  "license": "MIT",
  "dependencies": {
    "express": "^4.13.3"
  }
}
```

根据你系统的差异或者网络因素，你可能需要使用`sudo npm install express --save`，或者是`sudo cnpm install express --save`。

使用npm安装包的时候，除了`--save`选项，我们还可以使用`--save-dev`。两者在安装完包之后，都可以自动更新`package.json`文件，不同的是，前者更新的是`dependencies`字段，后者更新的是`devDependencies`字段。而`dependencies`和`devDependencies`的区别可以简单的理解成，前者是生产环境所需要的依赖，而后者是开发环境所需要的依赖。

在安装完毕express之后，我们需要新建一个启动文件，就是`package.json`中`main`字段指定的文件。当然你说我能不能新建一个文件叫别的什么名字，那当然也是没问题的。

```bash
> touch index.js
```

`index.js`文件的内容如下，

```javascript
var express = require('express');
var app = express();

app.get('/', function (req, res) {
    res.send('Hello world!');
});

app.listen(3000, function () {
    console.log('app is listening at localhost:3000');
});
```

最后再启动这个文件，

```bash
> node index.js
```

至此，我们使用express开发的一个最最简单的web程序就完成了。可以我浏览器输入`localhost:3000`，看看效果吧。


# 工作原理

经过上面的简单示例，看起来使用express做web真的非常简单。为什么可以做到这么简单呢？这是跟express的设计理念有关系的。

整个express应用可以简单的抽象成一系列的路由和中间件。当然这些路由和中间件之间存在着调用和先后关系。

那么，路由和中间件具体又是指的是什么呢？

简单来说，express中的路由可以看成一种**path watcher**，比如上面示例中的代码，

```javascript
app.get('/', function (req, res) {
    res.send('Hello world!');
});
```

这段代码中，express app将监控`/`路径，来自客户端所有对`/`路径的请求，都使用后面的回调函数处理。

而这里的处理请求的**回调函数**其实就是所谓的中间件。

所以，中间件其实就是**处理HTTP请求**的回调函数，一般来说它会有3个参数，分别是`req`，`res`和`next`，分别表示请求对象，响应对象以及next回调。next回调的作用是将控制权传递给下一个中间件。

如果一个中间件是专门用于错误处理的，它将会有4个参数，分别是`err`，`req`，`res`，`next`。其中第一个参数表示错误对象。

中间件的基本执行过程是这样的，**在中间件内部对req和res对象进行处理加工，执行相关逻辑，然后根据业务决定是否执行`next（）`函数，传递到下一个中间件**。

除此之外，一般我们建议一个中间件只专注做一件事，也就是说中间件的职责尽量单一，这样利于维护和协调中间件。

# 中间件机制

下面我们来详细的说一说express的中间件机制。

其实所谓的中间件就是一个函数回调。express中采用`use`来注册中间件。比如，

```javascript
var expresss = require('express');
var app = express();

app.use(function (req, res, next) {
    console.log('Time: ', new Date());
    next();
});

// more code...
```

如上例所示，这个中间件是整个app的第一个中间件，当有请求过来时，它将会被第一个调用，在console中打印出时间信息。执行完毕之后，通过`next()`将执行权传递给后面的中间件。

此外，我们可以再中间件内容做一些额外的事情，比如对请求路径的判断，

```javascript
app.use(function(req, res, next) {
    if (request.url === "/") {
        response.writeHead(200, { "Content-Type": "text/plain" });
        response.end("Welcome to the homepage!\n");
    } else {
        next();
    }
});

app.use(function(req, res, next) {
    if (req.url === "/about") {
        res.writeHead(200, { "Content-Type": "text/plain" });
        res.end("about page!\n");
    } else {
        next();
    }
});

app.use(function(req, res, next) {
    res.writeHead(404, { "Content-Type": "text/plain" });
    res.end("404 error!\n");
})
```

上面的代码表示，

- 如果请求的是`/`路径，返回给客户端`Welcome to the homepage!`。如果是其他路径，就调用下一个中间件。
- 第二个中间件中，判断路径是否是`/about`，如果是，将返回给客户端`about page!`。如果不是，调用下一个中间件。
- 当执行权传递到第三个中间件时，那么客户端请求的路径不是`/`也不是`/about`，此时我们将给出一个`404 error!`的提示，并同时终止中间件的继续调用。

从上面的示例代码，我们足以管中窥豹，领略一下express中间件机制，以及它们的调用策略。简单来说就是，**express按照顺序执行中间件，每个中间件做好自己的任务并决定是否调用下一个中间件**。

## 中间件的分类

express的官网上对中间件作了个分类，包含如下几种，

- [Application-level middleware](http://expressjs.com/guide/using-middleware.html#middleware.application)（应用级别中间件）
- [Router-level middleware](http://expressjs.com/guide/using-middleware.html#middleware.router)（路由级别中间件）
- [Error-handling middleware](http://expressjs.com/guide/using-middleware.html#middleware.error-handling)（错误处理中间件）
- [Built-in middleware](http://expressjs.com/guide/using-middleware.html#middleware.built-in)（内置中间件）
- [Third-party middleware](http://expressjs.com/guide/using-middleware.html#middleware.third-party)（第三方中间件）

应用级别的中间件其实就是指挂在`app`上的中间件。通常我们除了可以通过`app.use`来挂载中间件之外，express还提供了`app.METHOD`这种方式。这里的`METHOD`指的是http的方法。比如

- `app.get`
- `app.post`
- `app.put`

除此之外，还有一个特别的动词我们也可以使用，就是`app.all()`，它表示所有的请求都会通过这个中间件。看下面的示例代码，

```javascript
app.all("*", function(request, response, next) {
    response.writeHead(200, { "Content-Type": "text/plain" });
    next();
});

app.get("/", function(request, response) {
    response.end("Welcome to the homepage!");
});

app.get("/about", function(request, response) {
    response.end("Welcome to the about page!");
});

app.get("*", function(request, response) {
    response.end("404!");
});
```

所有的请求都将会执行第一个中间，用户设置http响应的头信息。如果请求的是`/`或者`/about`，则执行相应的中间件。这里注意，第二个和第三个中间件都没有调用`next()`，所以他们执行完毕之后，并不会调用后面的中间件。如果请求的路径不是`/`或者`/about`，那么第二个和第三个中间件都将不会被调用，第四个中间件将会被调用。

所以说，express应用中中间件的顺序是很重要的，它将影响中间件的执行顺序。


# Router机制

前面我们知道，可以通过`app.use`或者是`app.METHOD('/')`来配置路由。

在Express4中，路由成了一个单独的组件，express提供了一个新的对象`express.Router`，可以通过它来更好的配置路由。我们先来看一个示例，

```javascript
// birds.js
var express = require('express');
var router = express.Router();

router.use(function (req, res, next) {
    console.log('Time: ', new Date());
    next();
});

router.get('/', function (req, res) {
    res.send('Birds home page');
});

router.get('/about', function (req, res) {
    res.send('Birds about page');
});

module.exports = router;
```

```javascript
// index.js
var express = require('express');
var birds = require('./birds');

var app = express();

app.get('/', function (req, res) {
    res.send('app home page');
});

app.use('/birds', birds);

app.listen(3000, function () {
    console.log('server starting...');
});
```

通过这个简单的例子，我们可以独立封装一组路由成为一个路由中间件，在主文件中可以根据需要挂在到不同的路径上，如`app.use('/birds', birds)`。这样一来，路由中间件中对路由的配置最终会被解析成如下，

- `/birds/`
- `/birds/about/`

我们使用express进行web开发的时候，一般也是推荐建立一个routes文件夹，将所有页面的路径配置都抽象成一个路由中间件，然后在主文件中挂载。


## 模式匹配和参数

使用进行路由配置的时候，我们不仅仅可以使用字符常量，还可以使用字符串的模式匹配，如下，

```javascript
// will match acd and abcd
app.get('/ab?cd', function(req, res) {
    res.send('ab?cd');
});

// will match abcd, abbcd, abbbcd, and so on
app.get('/ab+cd', function(req, res) {
    res.send('ab+cd');
});

// will match abcd, abxcd, abRABDOMcd, ab123cd, and so on
app.get('/ab*cd', function(req, res) {
    res.send('ab*cd');
});

// will match /abe and /abcde
app.get('/ab(cd)?e', function(req, res) {
    res.send('ab(cd)?e');
});

// will match anything with an a in the route name:
app.get(/a/, function(req, res) {
    res.send('/a/');
});

// will match butterfly, dragonfly; but not butterflyman, dragonfly man, and so on
app.get(/.*fly$/, function(req, res) {
    res.send('/.*fly$/');
});
```

如你所见，路由的模式匹配其实很简单。你可以使用正则，让你的路由回调更加随心所欲的匹配不同的路径。

路由参数的意思就是你可以在路由配置时定义好路径中带有的参数，比如

```javascript
app.get('/book/:bookId', function (req, res, next) {
    console.log(req.params);
});
```

这样你就可以在回调中，处理`req`中的`params`对象。

## 路由回调

有时候我们可能会有这样一个需求，我需要在路由回调中进行一些预处理。比如下面这个例子

```javascript
var express = require('express');
var app = express();

app.get('/user/:id', function (req, res, next) {
    if (req.params.id === 0) {
        next('route');
    } else {
        next();
    }
}, function (req, res, next) {
    res.send('regular');
});

app.get('/user/:id', function (req, res, next) {
    res.send('special');
});

app.listen(3000);
```

上面的示例代码，有两点需要注意，

**1.**我们可以在路由配置的回调中，添加多个回调函数。如果有多个回调，我们可以有两种形式，如下，

```javascript
app.get('/', function() {}, function() {});

app.get('/', [callback1, callback2, callback3]);
```

**2.**同一个路由配置的多个回调函数在执行时可以被跳过。

上面代码中的`next('route')`表示**跳过**当前路由中间件中剩下的路由回调，执行下一个中间件。而`next()`表示执行当前中间件的下一个路由回调。所以，当我请求`/user/100`时，返回的是regular。当我请求`/user/0`时，返回的是special。

## `app.router()`

在express4中我们可以使用`app.router`对同一个路径做不同的请求方法配置，如下，

```javascript
app.route('/book')
    .get(function(req, res) {
        res.send('Get a random book');
    })
    .post(function(req, res) {
        res.send('Add a book');
    })
    .put(function(req, res) {
        res.send('Update the book');
    });
```

其实这种方式在路由中间件中也是可以用的。针对特定的路由定制场景是特别适用的，可以更好的模块化、节省一些冗余的代码。


# 模板引擎

之前我们的示例代码中，都是简单是使用`res.send`给客户端发送简单的文本。如果我们想要客户端渲染一个自定义的页面，那该怎么做呢？

这就需要用到模板了。严格来说，express使用的模板属于服务端模板，因为它是在服务端解析完毕，发送给客户端进行渲染的。express支持很多[模板引擎](https://github.com/strongloop/express/wiki#template-engines)，基本上将市面上常见模板引擎一网打尽了。express默认的模板引擎是[jade](https://github.com/jadejs/jade)，一款非常优雅的模板引擎。后面博主会有文章介绍jade。

言归正传，我们在express中如何使用模板引擎来加载模板呢？其实很简单，三步走即可，

1. 设置好模板的静态路径
2. 设置好渲染的模板引擎
3. 在需要的地方渲染你的模板

具体看下面的示例代码，

```javascript
// 省略无关代码

// 设置模板文件夹的路径
app.set('views', path.join(__dirname, 'views'));
// 设置模板文件的后缀名
app.set('view engine', 'jade');

app.get('/', function (req, res){
    res.render('index');
});
app.get('/about', function(req, res) {
    res.render('about');
});
app.get('/article', function(req, res) {
    res.render('article');
});

// 省略无关代码
```

我们在`views`目录下有3个模板，分别为`views/index.jade`，`views/about.jade`，`views/article.jade`，当请求不同的路径时，我们会发送不同的模板给前端渲染。

如果你使用的是别的模板引擎，请参阅模板引擎针对express的相关说明。

本文由于篇幅原因，不会关注express的方方面面，更多的内容请参考官网的[API Reference](http://expressjs.com/4x/api.html)




End! All rights reserved `@gejiawen`.


