postid: "about-seajs-and-nodejs"
title: Seajs与Nodejs的兼容
date: 2014-08-12 14:30:47
categories: [大前端]
tags: [转载, 前端规范]
---

[Sea.js](https://github.com/seajs/seajs)的模块遵循[CMD](https://github.com/seajs/seajs/issues/242)规范，与Node.js的模块规范非常相近，两者的模块可以很容易相互迁移。

本文系转载，版权归原作者所有，[点我看原文](https://github.com/seajs/seajs/issues/275)。


# 让Sea.js的模块跑在Node上

非常简单。先手需要安装seajs的Node模块：

```shell
$ npm install seajs -g
```

安装好后，在需要调用Sea.js模块的入口文件里，`require`下`seajs`：

**a.js**

```javascript
define(function(require, exports) {
    exports.name = 'A';
});
```

**main.js**

```javascript
require('seajs');

var a = require('./a');
console.log(a.name);
```

这样就可以在Node环境中运行Sea.js的模块了：

```shell
$ node main.js
A
```


# 让Node的模块跑在浏览器端

这个也很简单，将Node的模块封装成CMD模块即可：

**a.js**

exports.name = 'A';

封装成

```javascript
define(function(require, exports) {
    exports.name = 'A';
});
```

这样在浏览器端就可以通过Sea.js来加载使用了：

```javascript
seajs.use('./a', function(a) {
    console.log(a.name);
});
```


# 通用的前提条件

通过上面的例子可以看出，只要遵循CMD规范来书写模块代码，就可以非常方便的同时运行在浏览器端和服务器端。这是CMD中C字段的哈伊： **Common（通用）**。

这比RequireJS的AMD规范好很多。

但是，无论是CMD还是AMD规范，或者是未来的某个Modules/Cool规范，一个模块要想同时在不同环境下执行，就得遵循以下的前提条件：

1. **不能调用浏览器端的私有特性**，比如`attachEvent`之类的，除非你在Node端提前做好模拟。
2. **不能调用服务器端的私有特性**，比如`process`对象，除非你在浏览器端自己实现一个类似的`process`对象。
3. **只用不同规范中的交集**，比如CMD中有`module.url`属性，但Node中没有，要通用，就不能去调用这些有差异的借口。类似的，Node端的`__filename`在浏览器端不存在，要通用的话，也不能调用。


其实上面这些严格要求，是**非常自然的**。这就如同浏览器兼容一样，要写出所有浏览器下都能跑的代码，最好的方式是只用哪些所有浏览器都支持的特性，不然你就得用`if ... else ... `去搞了。


# 附录

Node.js与Sea.js在模块接口上的主要差异如下：

1. Node.js里，模块文件里的`this === module.exports`；而Sea.js里，`this === window`。
2. Node.js里，模块文件里的`return xxx`会被忽略；而Sea.js里，`return xxx`等价`module.exports = xxx`。
3. Node.js里，`require`是懒加载+懒执行的；而Sea.js里是提前加载好+懒执行。
4. Sea.js里，`require(id)`中的`id`必须是字符串直接量。Node.js里没有这个限制。


