title: NodeJS最佳实践
date: 2014-09-19 16:17:10
categories: [Nodejs]
tags: [nodejs, 翻译, nodejs-基础入门]
---

- 英文原文 [Node.js Best Practices](http://blog.risingstack.com/node-js-best-practices/?utm_source=nodeweekly&utm_medium=email)
- 译者 [gejiawen](https://github.com/gejiawen)

# 代码风格

## 回调函数的相关约定

所有的模块接口应该遵循 **优先暴露错误** 这一原则。如下代码所示，

```
module.exports = function (dragonName, callback) {
    // do some stuff here
    var dragon = createDragon(dragonName);

    // note, that the first parameter is the error
    // which is null here
    // but if an error occurs, then a new Error
    // should be passed here
    return callback(null, dragon);
}
```

## 在回调函数中总是优先检查错误

为了更好的理解这一原则，下面我们给出一个反例，

```
// this example is **BROKEN**, we will fix it soon :)
var fs = require('fs');

function readJSON(filePath, callback) {
    fs.readFile(filePath, function(err, data) {
        callback(JSON.parse(data));
    });
}

readJSON('./package.json', function (err, pkg) {
    // do something
}
```

当`fs.readFile`在执行抛出一个`Error`时，方法`readJSON`将不会按照代码的期望运行下去了。

我们对齐进行修正，代码如下，

```
// this example is **STILL BROKEN**, we are fixing it!
function readJSON(path, callback) {
    fs.readFile(filePath, function(err, data) {
        // here we check, if an error happened
        if (err) {
            // yep, pass the error to the callback
            // remember: error-first callbacks
            callback(err);
        }

        // no error, pass a null and the JSON
        callback(null, JSON.parse(data));
    });
}
```

这里，我们`fs.readFile`的回调中优先检查`err`对象，根据`err`的值进行不同的处理。

## 回调函数中的return

上面的代码仍然有一个问题，就是当回调中检查到`err`对象不为空时，代码运行完`if`分支之后并不会停下，而是继续运行下去。这是因为我们在相应的地方添加返回，这可能将会带来一些不可预料的异常。所以，一般来说，我们都会直接`return`回调。

```
// this example is **STILL BROKEN**, we are fixing it!
function readJSON(path, callback) {
    fs.readFile(path, function(err, data) {
        if (err) {
            return callback(err);
        }

        return callback(null, JSON.parse(data));
    });
}
```

## 只能同步方法中使用try-catch语句

此处有坑出没！当`JSON.parse`方法不能正确的把一个字符串解析成json对象时，它可能会抛出一个`exception`。

因为`JSON.parse`是一个同步方法，我们可以简单的使用一个`try-catch`语句块将其包装起来。如下所示，

```
// this example **WORKS**! :)
function readJSON(path, callback) {
    fs.readFile(path, function(err, data) {
        var parsedJson;

        // Handle error
        if (err) {
            return callback(err);
        }

        // Parse JSON
        try {
            parsedJson = JSON.parse(data);
        } catch (exception) {
            return callback(exception);
        }

        // Everything is ok
        return callback(null, parsedJson);
  });
}
```

**注意** 永远不要给异步方法（异步回调）添加`try-catch`，因为要捕获一个未来才会执行到的函数所抛出的错误是不可能的。

## 尽量避免使用`this`以及`new`

在Node.js中尝试绑定一个特定的上下文环境给运行环境不是好的主意，因为Node.js本身就是以使用回调函数及高阶函数为一大特色。它推荐的编程风格就是使用函数式的编程风格和思想。

当然，在有些场景下，使用基本原型、对象等编程手段将会更加有效率。但是如果可能的话，应该尽量避免这种思路，使用Node.js推荐的风格。

## 模块功能单一化

把问题分解成一个个简单的小问题是一个好的思路（the unix-way）。

> Developers should build a program out of simple parts connected by well defined interfaces, so problems are local, and parts of the program can be replaced in future versions to support new features.

始终应该遵循这样一个原则， **不要一个模块包含过多的功能，应该尽量保持简单和功能单一化** 。

## 使用成熟的异步模块

建议使用已有的成熟的异步处理模块来书写代码。

- [async](https://www.npmjs.org/package/async)
- [q](https://www.npmjs.org/package/q)
- ...

## 总是进行错误处理

*(译者：这部分的内容感觉好low，感觉没有存在的意义。)*

错误处理可以分为两大类， **操作性错误（operational errors）** 和 **代码错误（programmer errors）** 。

### 操作性错误

操作性错误可能发生在一些书写的很好的代码中，因为它们并不是bug，而是一些由于系统或者网络等原因导致的问题，比如，

- 请求超时
- 内存溢出
- 链接远程服务器失败
- ...

### 操作性错误的处理

下面是解决操作性错误的常规做法，

- 尝试解决导致错误的客观因素。如果文件丢失，那你应该先创建一个文件。
- 重复失败的操作。当网络连接异常的时候，应该多试几次。
- 告诉客户端，有些东西还没准备好。
- 重启相关的服务。

### 代码错误

代码错误就是程序bug！你可以通过如下几种方式来避免它们，

- 调用一个不带回调的异步方法（译者：这里啥意思？没懂！）
- 不要试图在`undefined`上读取一个属性

### 代码错误的处理

因为这些都是程序bug，将会导致你的程序直接崩溃，而且你还不知道何时何地将会导致你的程序溃崩。所以当这类错误发生导致程序崩溃时，我们使用一个守护进程重启你的程序。比如，[supervisord](http://supervisord.org/)或者[monit](http://mmonit.com/monit/)。


# 工作流程

## 使用`npm init`来开始一个新项目

`npm init`可以帮助你自动生成应用的`package.json`文件。它将某些字段设为默认值，当然你后面可以手动修改。

所以，当你开始一个新项目时，应该像下面这样，

```bash
mkdir my-awesome-new-project
cd my-awesome-new-project
npm init
```

## 提供`start`和`test`脚本

在你的`package.json`中你可以在`scripts`部分做一些脚本设置。`npm init`命令会默认生成`start`和`test`部分对`npm start`和`npm test`命令进行支持。

此外，你还可以自定义可运行的脚本设置， `npm run-script <SCRIPT_NAME>`。

## 环境变量

无论是生产环节还是开发环境的部署都应该带有相应的环境变量。一般性的做法是设置`NODE_ENV`环境变量。

使用[nconf](https://github.com/flatiron/nconf)模块你可以根据不同的环境加载你自定义的环境配置。

当然，你也可以使用内置的`process.env`对象来重置你的环境配置。

## 不要重复造轮子

建议优先使用已有的成熟的模块或者解决方案。[NPM](https://www.npmjs.org/)上面有大量的模块，总会找到你需要的模块的。

## 使用统一的代码风格

使用统一的代码风格，无论你的代码量多么的庞大，都是相对容易阅读的。统一的代码风格一般包含， *缩进规则* ， *变量命名规则* 等等。

这里有一份参考，[RisingStack](http://risingstack.com/)'s [Node.js Style Guide](https://github.com/RisingStack/node-style-guide)。


# 接下来...

我希望这篇文章能够让你在Node.js的世界中少走一些弯路。

balabalabalabala...

End. All rights reserved `@gejiawen`.
