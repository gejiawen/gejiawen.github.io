postid: "hexo-your-blog"
title: hexo-your-blog
date: 2014-06-19 10:46:57
updated: 2014-06-19 13:59:57
categories: [教程]
tags: [hexo, nodejs]
---

我曾纠结在哪里写博客，使用过的blog产品有oschina, cnblog这些博客站，还有过自己搭建博客。前者因为都现有的blog站点上写博客，有着很多的条条框框，不是太舒服。后者，由于没有自己的空间（穷屌丝，买不起服务器空间），所以折腾完就丢了。

直到我看到了[hexo](https://github.com/tommy351/hexo)，于是我喜欢上了这种使用markdown来写博客的方式，非常的爽。

[hexo](https://github.com/tommy351/hexo)是一个基于nodejs的静态博客生成程序，作者**[tommy351](http://twitter.com/tommy351)**。

这是一篇介绍关于如何使用[hexo](https://github.com/tommy351/hexo)搭建个人blog的教程。[参考链接](http://ibruce.info/2013/11/22/hexo-your-blog/)在此，文章版权归原作者所有。本blog参考了其中大部分内容。


> 更新：现在hexo已经升级到3.x.x版本，使用了core + plugin的设计思路，且代码使用promise风格重写了。所以本文的部分内容将不再使用hexo 3.x.x。

# 正文开始


## 环境要求

* git
* nodejs
* hexo


git与nodejs的安装不再赘述，hexo的安装如下：

```shell
npm install hexo
```


## 基本介绍
-------
[hexo](https://github.com/tommy351/hexo)目前最新版是2.7（截至这篇文章的发布时间），虽然仍然有不少的bug，但是在github上的人气相当可以，我相信它会越做越好的。

[这里](http://hexo.io/)是hexo的使用说明，下面是一些简明的使用：

```shell
hexo new [layout] "post_name"   // 新建一篇post
hexo generate                   // 生成静态文件
hexo deploy                     // 发布到托管网站上
```

hexo还比较人性化的提供了一些命令的缩写：

```shell
hexo n [layout] "post_name"
hexo g
hexo d
```

此外，`generate`和`deloy`操作还可以合并在一起：

```shell
hexo g -d
```

或者

```shell
hexo d -g
```

[这里](http://hexo.io/docs/commands.html)是一些命令的基本用法，下面只说一点关于`hexo new`的注意事项：

```shell
hexo new [layout] "post_name"
```

其中`layout`参数表示新建的post基于哪一种layout配置。那么，我们有哪些layout配置呢？默认带有了几种配置，见`scaffolds`文件夹。
你可以修改这些layout配置统一应用到新建post上。

你还可以在post中使用`<!-- more -->`来实现文章摘要的目的。如下：

```markdown
我是摘要
<!-- more -->
我是正文
```

## hexo主题

blog主题是一个很能体现个人特色的元素。话说萝卜青菜各有所爱，[这里](https://github.com/tommy351/hexo/wiki/Themes)是hexo的theme列表，你可以尝试选择一款自己喜欢的theme应用到自己的blog上。

安装theme的方法：

* 下载你喜欢的theme
* 放在`themes`文件夹中
* 修改hexo配置

```yml
theme: your_favourite_theme_name
```

如果你喜欢折腾，你还可以自定义属于自己的主题，一般来说，可以基于某一个现有的theme做一些修改。hexo的theme使用的是[ejs](http://embeddedjs.com/)模版。
使用hexo提供的[Variables](http://hexo.io/docs/variables.html)，在theme模版中作各种判断，就可以自定义自己的theme啦。


## hexo主题自定义修改

下面是一个添加[多说](http://duoshuo.com/)评论系统的例子。

我使用的是[light](https://github.com/hexojs/hexo-theme-light)主题，这个theme很淡雅，看起来还不错。
不过light默认使用的是disqus评论，可能在国内的速度不是很快，所以我最终选择了国产的多说。

* 首先修改主题的配置`_config.yml`，添加你的duoshuo帐号名称

    ```yml
    duoshuo_shortname: gejiawen-blog
    ```

* 然后修改相应的模版文件`layout/_partial/commnet.ejs`

*说明*，

我这里使用的主题是`light`，可能你并没有使用这个主题，那么就不一定是修改这个`comment.ejs`文件，可能这个文件都不一定存在了。具体要怎么办，那就需要自己摸索主题的各个模版文件是做什么的了。

添加相关代码：（在这之前你需要在多说上获取自己的通用代码）

```ejs
<% if (page.comments && theme.duoshuo_shortname){ %>
<section id="comment">
    <h1 class="title"><%= __('comment') %></h1>

    <!-- 多说评论框 start -->
    <div class="ds-thread" data-thread-key="<%= path %>" data-title="<%= page.title %>" data-url="<%= url %>"></div>
    <!-- 多说公共JS代码 start (一个网页只需插入一次) -->
    <script type="text/javascript">
        var duoshuoQuery = {short_name: "<%= theme.duoshuo_shortname %>"};
        (function () {
            var ds = document.createElement('script');
            ds.type = 'text/javascript';
            ds.async = true;
            ds.src = (document.location.protocol == 'https:' ? 'https:' : 'http:') + '//static.duoshuo.com/embed.js';
            ds.charset = 'UTF-8';
            (document.getElementsByTagName('head')[0]
                    || document.getElementsByTagName('body')[0]).appendChild(ds);
        })();
    </script>
</section>
<% } %>
```

*一点解释*

* `page.comments`: 是用于判断当前页面是不是用于文章页面，即当处于首页或者归档这个页面时，这个条件是不满足的。
* `theme.duoshuo_shortname`: 你在主题配置中设置的多说用户名。
* `<%= path %>`: 将当前文章的path作为多说的thread-key，这个变量要求唯一，所以这里我用了path。


