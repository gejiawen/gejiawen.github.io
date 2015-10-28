postid: 'gitbook-fast-started'
title: Gitbook快速指南
categories: [教程]
tags: [gitbook]
date: 2015-10-28 23:11:12

---

[Gitbook](https://github.com/GitbookIO/gitbook)是一款利于git及markdown快速构建在线书籍的工具。官网是[www.gitbook.com](http://www.gitbook.com)。其实Gitbook自身也是一个提供书籍在线发布的平台。不过由于一些网络的因素，gitbook平台在国内的访问速度和稳定性堪忧。

本文将简要的叙述如何使用Gitbook，基于github来快速构建一个在线书籍。我的博客上就有一个gitbook应用的[示例](http://gejiawen.github.io/coding-book/)，可以先看看效果。

# 安装

依托Node.js平台，安装gitbook是非常简单的。

```shell
npm install -g gitbook-cli
```

`gitbook-cli`是gitbook的命令行操作工具，它封装了一系列的命令，包括书籍初始化、预览、构建等等操作。

第一次安装`gitbook-cli`之后，还会自动安装最新版的gitbook应用程序。我们之前说`gitbook-cli`只是一个命令行工具，仅有`gitbook-cli`还不够，我们还需要安装gitbook程序。

由于网络因素，可能第一次安装gitbook会较慢，因为它的安装过程是先下载gitbook的源码，然后在本地编译。这个编译过程是依赖node-gyp的。

在安装gitbook完成之后，我们可以使用`gitbook versions`来查看当前已经安装的gitbook版本，

```shell
gitbook versions
```

结果如下，

![](/res/gitbook快速指南/001.png)

我的机器上目前安装了两个版本的gitbook，当前正在使用的是`2.5.1`版本。

除此之外，我们还有通过`gitbook versions:available`来查看当前gitbook的所有版本，还可以通过`gitbook versions:update`升级到最新版本的gitbook。


# 构建

因为我们要基于github来构建在线书籍，所以首先我们得创建一个github repo。创建好repo之后，我们再`git clone`到本地。

```bash
git clone YOUR_REPO_URL
cd YOUR_REPO_FOLDER
```

然后通过`gitbook init`来初始化一个书籍。

```bash
➜  test-gitbook  gitbook init
info: init book at /Users/gejiawen/code/test-gitbook
info: detect structure from SUMMARY (if it exists)
info: create SUMMARY.md
info: create README.md
info: initialization is finished

Done, without error
```

成功初始化之后，会在工作目录中生成两个markdown文件，`SUMMARY.md`和`README.md`文件。这两个文件的作用如下，

- `SUMMARY.md`文件是书籍的目录结构
- `README.md`文件是书籍的扉页（介绍）

这两个文件缺一不可。

其中我们可以编辑`SUMMARY.md`文件来自定义自己书籍的目录结构，比如

```markdown
- [Chapter 1](ch1/index.md)
    - [1-1](ch1/1-1.md)
    - [1-2](ch1/1-2.md)
- [Chapter 2](ch2/index.md)
    - [2-1](ch2/2-1.md)
    - [2-2](ch2/2-2.md)  
```

书籍的每一页内容都可以使用markdown来编辑。怎么样，是不是很简单。

在gitbook 2.0以后，我们还可以在书籍目录中引入`book.json`来配置书籍。

一个`book.json`文件的内容大概如下，

```json
{
    "gitbook": "2.x.x",
    "title": "蛋糕仙人的编码生活",
    "description": "蛋糕仙人的编码生活",
    "structure": {
        "readme": "intro.md",
        "summary": "summary.md"
    }
}
```

其中`gitbook`字段表示的是构建书籍时所采用gitbook程序版本。这里官方推荐使用`"2.x.x"`，这种模糊的版本号可以适配gitbook 2的所有子版本，这无疑是一种更优的做法。

而`structure`表示自定义关键页面的源地址。比如，可能你是那种一个项目必带`README.md`文件的强迫症患者，但是gitbook应用中默认采用`README.md`文件作为书籍的扉页。你可能会觉得这不太合理，此时我们就可以使用`structure`来自定义readme的地址啦。

对`book.json`欲了解更多内容，请移步官方的[帮助文档](http://help.gitbook.com/format/configuration.html)。

到此为止，一个在线书籍基本上已经ready了。接下来我们在本地预览一下效果，

```bash
gitbook serve
```

待服务启动后，可以使用浏览器打开`localhost:4000`在本地查看效果了。嗯，简约的风格，我很喜欢。

# 部署

在完成书籍的写作之后，我们可以通过`gitbook build`来构建书籍源文件，嗯，其实都是一些html文件啦。

```bash
➜  test-gitbook  gitbook build
info: loading book configuration....OK
info: load plugin gitbook-plugin-highlight ....OK
info: load plugin gitbook-plugin-search ....OK
info: load plugin gitbook-plugin-sharing ....OK
info: load plugin gitbook-plugin-fontsettings ....OK
info: >> 4 plugins loaded
info: start generation with website generator
info: clean website generatorOK
info: generation is finished

Done, without error
```

`gitbook build`命令默认会在书籍目录下生成一个`_book`文件夹。书籍的源文件就在里面啦。

如果你有自己的服务器，你就将`_book`下的所有文件拷贝到你自己的服务器上就行了。

如果你没有自己的服务器（比如我这种穷人），没关系，我们可以依托github来发布。

过程如下，

新建一个新的分支，叫做`gh-pages`，然后在`gh-pages`分支中仅提交`_book`下的文件即可，然后将此分支提交github。这样github就会自动以[Jekyll](https://github.com/jekyll/jekyll)的形式部署你的在线书籍了。

题外话，我是一个轻度强迫症患者。我博客上的gitbook[示例](http://gejiawen.github.io/coding-book/)中，其markdown源代码我是在master分支上，而`gitbook build`的html源代码在gh-pages分支上。

在markdown源代码中，我将所有的md文件都放在了`_source`文件夹中，在构建的时候使用如下命令，

```bash
gitbook build _source _book
```

然后将生成的html源代码（即`_book`的内容）再提交到`gh-pages`分支，这样就达到了在一个repo中使用两个分支来独立的管理md源文件和html源代码了。具体的可以参考示例的[repo](https://github.com/gejiawen/coding-book)。

# 总结

gitbook总得来说还是一个非常给力的工具，特别是使用markdown这种简约的语言来写作，真是深得我心啊。有了gitbook和github，老是再也不担心我的写作了。所以，有创作热情的同学们，赶紧使用起来吧。

