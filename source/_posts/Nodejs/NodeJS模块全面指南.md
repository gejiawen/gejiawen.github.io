title: NodeJS模块全面指南
date: 2015-03-17 18:16:24
categories: [Nodejs]
tags: [nodejs, nodejs-基础入门]

---


# NodeJS模块

所谓的NodeJS模块其实就是指NodeJS package，即nodejs包。

在使用NodeJS进行开发的时候，往往需要用到各种各样的第三方包。当然，很多时候我们在实际开发的时候自己也会按照功能或者需求来封装一个本地的包。

那么问题来了，NodeJS模块究竟是什么？

## 什么是NodeJS模块？

在说这个问题之前，我们有必要先提出一个概念，即**模块规范**。关于模块规范可以参与之前的这篇文章[从CommonJS到Sea.js](http://gejiawen.github.io/2014/08/12/%E5%A4%A7%E5%89%8D%E7%AB%AF/%E4%BB%8ECommonJS%E5%88%B0Seajs/)。

所以，现阶段Javascript领域大体上有三种比较流行的模块规范，一种是AMD规范，一种是CMD规范，还有一种就是CommonJS规范。NodeJS采用的规范就是CommonJS规范。这三种规范中，前两者专注于客户端，即浏览器端的规范标准，而后者而其实是服务器端的规范。

CommonJS规范说，一个单独的文件其实就是一个模块。在NodeJS中，一个模块可以是一个单独的文件，也可以是一个包含多个文件（子模块）的目录。与此同时，CommonJS规范还要求模块都采用统一的格式`exports`或者`module.exports`导出模块接口。

## NodeJS包和模块

上面我们已经知道了模块的概念，那么包其实就是**包含多个模块目录**。同时，还要附加一个重要的`package.json`文件。那么这个`package.json`文件是个什么情况呢？如下图，

![](img-1-2-01.png)

这个json文件可以配置的很简略，也可以配置的很复杂。关于各个字段的具体含义可以参阅[官方文档](https://docs.npmjs.com/)或者[汉化文档](http://mujiang.info/translation/npmjs/files/package.json.html)。

其实`package.json`就是用来声明NodeJS包的，包括包名字，版本，作者，包入口信息，依赖等信息。

其中有两个字段这里稍微提一下，`dependencies`和`devDependencies`字段。前者表明包在生产环境需要的第三方依赖，而后者表明包在开发阶段所需要的第三方依赖（比如构建、测试等第三方包等）。

# npm包管理器

[NPM](https://www.npmjs.com/)是NodeJS包管理器，可以看成类似Java中的Maven或者Ruby中gem。

现在的npm是在最初的版本上改版而来的，界面更好看了，对包信息的展示更加人性化了。而且目前除了用于面向普通开发者提供服务之外，还提供私有包仓库和企业级包仓库。听说马上再一次改版的npm3.0也要到来了，提供了众多的优化和新特性，详情请参阅[这篇文章](http://www.tuicool.com/articles/6nM7Bf)。

## 创建包

前面我们说`package.json`是NodeJS包的标识或者说配置文件。所以，任何一个NodeJS包都是从新建`package.json`文件开始的。

这里我们一般不会傻傻的手动去新建一个`package.json`文件，而是通过npm工具来生成。

```bash
$ cd YOUR_PROJECT_DIR
$ npm init
```

在命令行中键入`npm init`后，CLI将会出现一些互动的提示来引导你完成`package.json`的生成。如下图，

![](img-2-1-01.png)

当然CLI只会询问你一些`package.json`文件必须的字段，而且它会智能的给出一些默认值（括号中的内容），最后它会向你确认是否可行。

`package.json`中有一个字段为`main`，此字段为意为包的入口。如果留空的话，NodeJS会默认将包目录下的`index.js`作为入口文件。

## 导出包接口

NodeJS模块使用的是CommonJS规范，使用`exports`或者`module.exports`导出模块准备暴露的接口。比如，

```javascript
exports.sayHello = function() {
    // your code
};

exports.sayGoodbye = function() {
    // your code
};
```

或者，

```javascript
module.exports = function People() {
    // your code
};
```

那么，`exports`和`module.exports`这两种方式到底有什么区别呢？

其实在NodeJS内部，模块真正返回的是`module.exports`对象，而`exports`只是`module.exports`的引用而已。如下，

```javascript
exports = module.exports = {};
```

如果你直接赋值一个函数（`function`）或者一个对象（`{}`）给`exports`，这样的话就破坏了`exports`和`module.exports`的引用关系了，而模块将会返回空对象。所以当我们需要暴露一个函数或者一个对象时，应该直接赋值给`module.exports`而不是`exports`。

本博客前面有一篇文章[如何导出NodeJS模块](http://gejiawen.github.io/2014/10/16/Nodejs/%E5%A6%82%E4%BD%95%E5%AF%BC%E5%87%BANodeJS%E6%A8%A1%E5%9D%97/)就是阐述NodeJS如何返回接口的。

## 发布包

好了，在我们完成模块的编写之后。我们希望将自己的包发布到NPM上，成就自己的同时又方便他人。该怎么做呢？

其实很简单。npm同样提供了相应的接口，**[npm adduser](https://docs.npmjs.com/cli/adduser)**和**[npm publish](https://docs.npmjs.com/cli/publish)**。

```bash
$ npm adduser
```

npm会向你索要[npmjs.com](https://www.npmjs.com/)用户名，密码以及Email。如果没有问题，接下来就可以执行`npm publish`了。

这里提一下执行`npm publish`时经常会遇到的两个问题。

第一，提示你没有权限发布。这种情况往往是你的包名已经在npm仓库中被占用了。所以这种情况你需要给你的包换个名字。
第二，提示你必须更新版本。这种情况一般是你是在本地更新了包，然后想更新npm仓库时却出现了这种错误。造成这个错误的原因其实很简单，因为已经发布在npm仓库中的包不允许不改变版本号的情况下就改变包代码。所以这种情况你需要改动`package.json`中的`version`字段。

## 额外的问题

我们说整个NodeJS社区都是一片欣欣向荣的景象，npm仓库的包数量很多，社区也很活跃。这些都是好现象，而且我个人也非常看好NodeJS在未来的发展。

但是，就我个人使用NodeJS的经验，遇到两个可能存在一些隐患的地方。

第一，npm仓库上的第三方包质量参差不齐，有的包基本就是垃圾，给使用者可能会造成一些损失。
第二，有些第三方包本身升级了，但是其周边插件包的更新却跟不上，有时候你为了向插件包妥协，不得不放弃新版本而使用低版本。
第三，因为第三方包是可以直接在服务器端运行的，有时候可能需要考虑一些安全因素。


# 创建命令行工具包

通过`npm install -g xxx`命令安装的部分第三方包后，就可以直接在命令行上运行相关命令了。比如下图，

![](img-3-01.png)

这种效果是如何做的呢？

## 添加`bin`字段

我们可以在`package.json`文件中添加一个叫做`bin`的字段。比如，npm cli中就是这样的，

```json
{
    "bin": {
        "npm" : "./cli.js"
    }
}
```

然后你在命令行中输入`npm`，NodeJS就会去执行对应的NodeJS模块。

如果你只有一个可执行命令，并且还和包名一致，那么`package.json`中你可以这么写，

```json
{
    "name": "my-package",
    "version": "1.2.5",
    "bin": "./bin/my-package"
}
```

然后，你需要在`bin/my-package`文件的第一行添加如下代码，

```bash
#!/usr/bin/env node
```

## 命令行参数的解析

我们通过命令行使用部分第三方包时，有时候包会提供各种命令和参数，如下图，

![](img-3-2-01.png)

从图中可以看出，这个[harp](http://harpjs.com/)提供四个命令以及两个参数，并且简略的展示了Usage。

那么这种比较完备的命令行包是如何做的呢？

这里我们一般会有两个方案去做成这件事，第一就是使用原生的`progress.argv`来解析，另一种方案就是使用[commander.js](https://www.npmjs.com/package/commander)。后面我会专门写篇文章来阐述`commander.js`的用法。

# 参考示例

[bullhead](https://www.npmjs.com/package/bullhead)是作者随便写的一个小玩意儿，主要目的是用来联系npm包创建及发布等操作。有兴趣可以参考。

# 参考列表

- [npmjs.com](https://www.npmjs.com/)
- [快速创建基于npm的nodejs库](http://blog.fens.me/nodejs-npm-package/)

End. All rights reserved `@gejiawen`.


