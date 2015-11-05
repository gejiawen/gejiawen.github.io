postid: "chrome-plugin-dev-guide"
title: "Chrome插件开发简要指南"
date: 2015-08-18 14:48:49
categories: [Javascript]
tags: [chrome]

---

Chrome是个非常牛逼的浏览器，速度快，界面清爽，对开发人员友好。而且提供了种类齐全且高质的插件。今天我们就来简单聊一聊如何开发chrome插件。

# 基本知识

在具体的开发之前，我们先来科普一些基础知识。

现阶段的chrome的应用商店中，将插件（这里说的插件指代chrome应用的所有分类，下文亦同）分为如下几种，包括**应用**、**游戏**、**扩展程序**、**主题背景**。而且插件的种类多种多样，保罗万象，基本上都能找到你想要的功能的插件。

## 文件结构

在应用商店中下载下来的插件基本上都是以`.crx`为文件后缀，该文件其实就是一个压缩包，包括插件所需要的html、css、javascript、图片资源等等文件。

下图是[FEHelper](http://www.baidufe.com/fehelper)的插件包解压后的文件，

![](/res/chrome-plugin-dev-guide/001.png)

其中，

- `manifest.json`是整个插件的功能及文件配置清单，非常重要。
- `static`目录是放置整个插件的静态资源文件的，包括css、js、图片等等资源
- `template`目录是放置整个插件的功能页面模板的。
- `_locales`目录是放置整个插件的国际化语言脚本的。

一般来说，清单文件`manifest.json`文件是必须的，且必须放在插件开发目录的根目录上。其他的目录都可以自定义。

这个插件安装好的用户界面如下图，

![](/res/chrome-plugin-dev-guide/002.png)

图中的每一个链接都是一个功能项，点击后将会打开一个chrome-extension下的页面，url类似这样，

```
chrome-extension://pkgccpejnmalmdinmhkkfafefagiiiad/template/fehelper_endecode.html
```

而这些页面其实就是我们之前看到的`template`中的模板文件。

## `manifest.json`文件

清单文件的作用是提供插件的各种信息，例如插件能够做的事情，以及插件的文件配置等等信息。下面是一个清单文件的示例，

```json
{
  "name": "我的扩展程序",
  "version": "2.1",
  "manifest_version": 2,
  "description": "从Google获得信息。",
  "icons": {
    "128": "icon_128.png"
  },
  "background": {
    "persistent": false,
    "scripts": ["bg.js"]
  },
  "permissions": ["http://*.google.com/", "https://*.google.com/"],
  "browser_action": {
    "default_title": "",
    "default_icon": "icon_19.png",
    "default_popup": "popup.html"
  }
}
```

`mianfest.json`文件每个字段都有其特殊的含义，[这份文档](http://chrome.liuyixi.com/manifest.html)有对清单文件的每个字段进行解释。

## 两种用户界面

chrome插件一般有两种用户界面，一种是**浏览器按钮**，一种是**页面地址栏按钮**。前者称之为**Browser Actions**，后者被称之为**Page Actions**。如下图，

![](/res/chrome-plugin-dev-guide/003.png)

*Page Actions* 与 *Browser Actions* 的区别就是 *Page Actions* 并不是在每个页面上都是有用的，它必须在特定的页面中插件功能才能使用。

这两种用户面界面分别对应`manifest.json`中的`browser_action`和`page_action`字段。


## 弹出窗口和后台页面

弹出窗口一般用于插件和用户的交互，而后台页面一般用于插件本身做一些额外的事情。比如有时候，插件需要联网进行数据同步等操作，这种操作用户是无感知的，所有就要求插件需要有一个后台页面来运行这部分的逻辑。

其实后台页面还可以分为[持久运行的后端页面](http://chrome.liuyixi.com/background_pages.html)和[事件页面](http://chrome.liuyixi.com/event_pages.html)，这里对这两者就不多做说明了，更多的内容可以参阅具体的文档。

## 其他

chrome插件的相关知识，远远不止上面介绍的几点，比如邮件菜单、插件与浏览器之间的交互、内容安全（CSP）、消息传递、国际化、部署与托管等等方面，由于篇幅原因，这里将不会作过多的描述。更多的内容请参阅相关文档，[这里](http://chrome.liuyixi.com/docs.html)。

# 自己做一个

有了上面的基础知识之后，我们就可以尝试自己来做一个chrome插件玩玩了。

首先我们新建一个文件夹叫做`todo-plugin`，作为插件开发的工作目录，然后在目录中新建清单文件，`manifest.json`文件内容如下，

```json
{
    "name": "todo-plugin",
    "version": "0.9.0",
    "manifest_version": 2,
    "description": "chrome plugin demo",
    "browser_action": {
        "default_icon": "icon.png",
        "default_title": "Todo List",
        "default_popup": "popup.html"
    }
}
```

在清单文件中，我们定义了插件的名称、版本、描述，插件的浏览器按钮行为以及插件所需要注入的脚本文件。按钮行为中还包括了默认的按钮图标、按钮标题（鼠标悬停时显示）以及默认的弹出框`popup.html`。

**注意**：新版的chrome插件开发需要在清单文件中制定`mainfest_version`为2。

这里的弹出框其实就是我们这个插件与用户交互的主要界面，它其实就是一个html页面。不过这个`popup.html`文件并不需要`<html></html>`、`<body></body>`、`<head></head>`这样的标签。`popup.html`文件内容如下，

```html
<style>
    body {
        width: 150px;
    }
    #add-new-item {
        cursor: pointer;
        color: #CCC;
    }

    .hide {
        display: none;
    }

    .show {
        display: block;
    }

    .item {
        cursor: pointer;
        margin: 5px 0;
    }

    .item input {
        display: inline-block;
        width: 12px;
        height: 12px;
    }

    input {
        width: 120px;
    }
</style>


<div id="add-new-item">添加新项</div>
<div id="add-new-item-input" class="hide">
    <input type="text" id="new-item-title" placeholder="添加新任务"/>
</div>
<div id="item-list"></div>


<script type="text/javascript" src="main.js"></script>
```

这个html文件其实很简单（chrome插件的html页面往往都非常简单，都是一些列表的展示，轻量的输入控件等等），而且为了简便，我把弹出窗的样式直接写在html中了。

最后一行我们使用外链的方式加载了popup页面的交互逻辑。`main.js`内容也很简单，如下，

```javascript
(function () {
    var $ = function (id) {
        return document.getElementById(id);
    };

    var Tasks = {
        show: function (obj) {
            obj.className = '';
            return this;
        },
        hide: function (obj) {
            obj.className = 'hide';
            return this;
        },
        $addNewItem: $('add-new-item'),
        $addNewItemInput: $('add-new-item-input'),
        $itemList: $('item-list'),
        $newItemTitle: $('new-item-title'),

        init: function () {
            //打开添加文本框
            Tasks.$addNewItem.addEventListener('click', function () {
                Tasks.show(Tasks.$addNewItemInput).hide(Tasks.$addNewItem);
                Tasks.$newItemTitle.focus();
            }, true);
            //回车添加任务
            Tasks.$newItemTitle.addEventListener('keyup', function (ev) {
                var ev = ev || window.event;
                if (ev.keyCode == 13) {
                    //TODO:写入本地数据
                    var task = Tasks.$newItemTitle.value;
                    Tasks.AppendHtml(task);
                    Tasks.$newItemTitle.value = '';
                    Tasks.hide(Tasks.$addNewItemInput).show(Tasks.$addNewItem);
                }
                ev.preventDefault();
            }, true);
            //取消添加
            Tasks.$newItemTitle.addEventListener('blur', function () {
                Tasks.$newItemTitle.value = '';
                Tasks.hide(Tasks.$addNewItemInput).show(Tasks.$addNewItem);
            }, true);
            //TODO 初始化数据，加载本地数据，生成html
        },
        //增加
        Add: function () {
            //TODO
        },
        //修改
        Edit: function () {
            //TODO
        },
        //删除
        Del: function () {
            //TODO
        },
        AppendHtml: function (title) {
            var oDiv = document.createElement('div');
            oDiv.className = 'item item-todo';
            var oInput = document.createElement('input');
            oInput.type = 'checkbox';
            var oTitle = document.createElement('span');
            oTitle.innerHTML = title;
            oDiv.appendChild(oInput);
            oDiv.appendChild(oTitle);
            Tasks.$itemList.appendChild(oDiv);

            oDiv.addEventListener('click', function () {
                //TODO
            }, true);
        },
        RemoveHtml: function () {
            //TODO
        }
    }
    Tasks.init();
})();
```

在这段js中，我们声明了一个`Tasks`对象，这个对象包括了一系列的功能方法，当然有部分未实现，不过核心功能都实现了。在`Tasks.init`方法中，我们给一个操作实体（一个div元素）注册了点击事件，当触发点击事件时，我们将会展示一个输入框让用户输入任务描述，通过回车键来添加用户刚输入的任务。好了，这就是`todo-plugin`插件的基本功能。下面是本插件的效果预览，

![](/res/chrome-plugin-dev-guide/004.png)

# 小结

好了，至此我们总算完成一个简陋的chrome插件的开发了。功能虽然简陋，但是基本上所有的chrome插件开发都是遵循这条路。本文示例的代码中有许多的`TODO`，其实我们可以将这个todo插件做的更加复杂，比如可以采用localstorage来存储数据，甚至可以使用网络来存储数据。至于更多的内容，本文就不再作过多的阐述了，因为本文毕竟是**简要指南**嘛😎😂

# 参考链接

- [chrome插件中文开发文档(非官方)](http://chrome.liuyixi.com/overview.html)
- [手把手教你开发Chrome扩展](http://www.cnblogs.com/walkingp/archive/2011/03/31/2001628.html)


