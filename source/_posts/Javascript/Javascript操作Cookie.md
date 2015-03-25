title: Javascript操作Cookie
date: 2014-07-18 17:43:40
categories: [Javascript]
tags: [javascript, 转载]
---

本文系转载，但是找不到原文的地址了。:(

# 什么是Cookie


“cookie是存储于访问者的计算机中的变量。每当同一台计算机通过浏览器请求某个页面时，就会发送这个 cookie。你可以使用 JavaScript 来创建和取回cookie的值。” - w3school

cookie是访问过的网站创建的文件，用于存储浏览信息，例如个人资料信息。

从JavaScript的角度看，cookie就是一些字符串信息。这些信息存放在客户端的计算机中，用于客户端计算机与服务器之间传递信息。

在JavaScript中可以通过`document.cookie`来读取或设置这些信息。由于cookie多用在客户端和服务端之间进行通信，所以除了JavaScript以外，服务端的语言（如PHP）也可以存取cookie。


# Cookie基础知识

* cookie是有大小限制的，每个cookie所存放的数据不能超过4kb，如果cookie字符串的长度超过4kb，则该属性将返回空字符串。
* 由于cookie最终都是以文件形式存放在客户端计算机中，所以查看和修改 cookie 都是很方便的，这就是为什么常说cookie不能存放重要信息的原因。
* cookie是存在有效期的。在默认情况下，一个cookie的生命周期就是在浏览器关闭的时候结束。如果想要cookie能在浏览器关掉之后还可以使用，就必须要为该cookie设置有效期，也就是cookie的失效日期。
* `alert(typeof document.cookie)`结果是`String`。
* cookie有域和路径这个概念。**域就是domain的概念**，因为浏览器是个注意安全的环境，所以不同的域之间是不能互相访问 cookie的(当然可以通过特殊设置的达到cookie跨域访问)。**路径就是routing的概念**，一个网页所创建的cookie只能被与这个网页在同一目录或子目录下得所有网页访问，而不能被其他目录下得网页访问。
* 其实创建cookie的方式和定义变量的方式有些相似，都需要使用cookie名称和cookie值。同个网站可以创建多个cookie，而多个cookie可以存放在同一个cookie文件中。


# Cookie常见问题

* cookie 存在两种类型：
    * 你浏览的当前网站本身设置的cookie。
    * 来自在网页上嵌入广告或图片等其他域来源的第三方cookie(网站可通过使用这些cookie跟踪你的使用信息)。
* 刚刚基础知识里面有说到cookie生命周期的问题，其实cookie大致可分为两种状态：
    * 临时性质的cookie。当前使用的过程中网站会储存一些你的个人信息，当浏览器关闭后这些信息也会从计算机中删除。
    * 设置失效时间的cookie。就算浏览器关闭了，这些信息业依然会在计算机中。如登录名称和密码，这样无须在每次到特定站点时都进行登录。这种cookie可在计算机中保留几天、几个月甚至几年。
* cookie 有两种清除方式：
    * 通过浏览器工具清除cookie(有第三方的工具，浏览器自身也有这种功能)。
    * 通过设置cookie的有效期来清除 cookie。
    * 注：**删除cookie有时可能导致某些网页无法正常运行**
* 浏览器可以通过设置来接受和拒绝访问cookie。
* 出于功能和性能的原因考虑，建议尽量降低cookie的使用数量，并且要尽量使用小cookie。
* 关于cookie编码的细节问题将会在cookie高级篇中单独介绍。
* 假如是本地磁盘中的页面，chrome的控制台是无法用JavaScript读写操作cookie的，解决办法...换一个浏览器^_^。

# Cookie基础用法


## 简单的存取操作

在使用JavaScript存取cookie时，必须要使用Document对象的cookie属性。
一行代码介绍如何创建和修改一个cookie，

```javascript
document.cookie = 'username=Darren';
```

以上代码中`username`表示cookie名称，`Darren`表示这个名称对应的值。假设cookie名称并不存在，那么就是创建一个新的cookie；如果存在就是修改了这个cookie名称对应的值。如果要多次创建cookie，重复使用这个方法即可。


## cookie的读取操作

要精确的对cookie进行读取其实很简单，就是对字符串进行操作。从w3school上copy这段代码来做分析，

```javascript
function getCookie(c_name) {
    // 先查询cookie是否为空，为空就return
    if (document.cookie.length > 0) {　　
        // 通过String对象的indexOf()来检查这个cookie是否存在，不存在就为 -1
        c_start = document.cookie.indexOf(c_name + "=")；

        if (c_start !== -1) {
            //最后这个+1其实就是表示"="号啦，这样就获取到了cookie值的开始位置
            c_start = c_start + c_name.length + 1;　　
            // 其实我刚看见indexOf()第二个参数的时候猛然有点晕，后来想起来表示指定的开始索引的位置...这句是为了得到值的结束位置。因为需要考虑是否是最后一项，所以通过";"号是否存在来判断
            c_end=document.cookie.indexOf(";",c_start);

            if (c_end === -1) ｛
                c_end = document.cookie.length；
            }
            // 通过substring()得到了值。想了解unescape()得先知道escape()是做什么的，都是很重要的基础，想了解的可以搜索下，在文章结尾处也会进行讲解cookie编码细节
            return unescape(document.cookie.substring(c_start,c_end));
        }
        return "";
    }
}
```

当然想实现读取cookie的方法还有不少，比如数组，正则等，这里就不往细说了。


## 设置cookie的有效期

文章中常常出现的cookie的生命周期也就是有效期和失效期，即cookie的存在时间。在默认的情况下，cookie会在浏览器关闭的时候自动清除，但是我们可以通过expires来设置cookie的有效期。语法如下，

```javascript
document.cookie = "name=value;expires=date";
```

面代码中的date值为GMT(格林威治时间)格式的日期型字符串，生成方式如下：

```javascript
var date = new Date();
date.setDate(date.getDate() + 30);
date.toGMTString();
```

上面三行代码分解为几步来看，

* 通过`new`生成一个Date的实例，得到当前的时间
* `getDate()`方法得到当前本地月份中的某一天，接着加上30就是我希望这个cookie能过在本地保存30天
* 接着通过`setDate()`方法来设置时间
* 最后用`toGMTString()`方法把Date对象转换为字符串，并返回结果

通过下面这个完整的函数来说明在创建 cookie 的过程中我们需要注意的地方 - 从w3school复制下来的。创建一个在cookie中存储信息的函数，

```javascript
function setCookie(c_name, value, expiredays) {
    var exdate=new Date();
    exdate.setDate(exdate.getDate() + expiredays);
    document.cookie = c_name + "=" + escape(value) + ((expiredays === null) ? "" : ";expires=" + exdate.toGMTString());
}

// 调用
setCookie('username','Darren', 30)；
```

现在我们这个函数是按照天数来设置cookie的有效时间，如果想以其他单位（如：小时）来设置，那么改变第三行代码即可：

```javascript
exdate.setHours(exdate.getHours() + expiredays);
```

# Cookie高级篇


## cookie路径概念

在基础知识中有提到cookie有域和路径的概念，现在来介绍路径在cookie中的作用。

cookie一般都是由于用户访问页面而被创建的，可是并不是只有在创建cookie的页面才可以访问这个cookie。

默认情况下，只有与创建cookie的页面在同一个目录或子目录下的网页才可以访问，这个是因为安全方面的考虑，造成不是所有页面都可以随意访问其他页面创建的cookie。举个例子，

> 在"http://www.cnblogs.com/Darren_code/"这个页面创建一个cookie，那么在"/Darren_code/"这个路径下的页面如："http://www.cnblogs.com/Darren_code/archive/2011/11/07/Cookie.html"这个页面默认就能取到cookie信息。

可在默认情况下，"http://www.cnblogs.com"或者"http://www.cnblogs.com/xxxx/"就不可以访问这个cookie（光看没用，实践出真理^_^）。

那么如何让这个cookie能被其他目录或者父级的目录访问类，通过设置cookie的路径就可以实现。例子如下，

```javascript
    document.cookie = "name=value;path=path";
    document.cookie = "name=value;expires=date;path=path";
```

path就是cookie的路径。

最常用的例子就是让cookie在跟目录下，这样不管是哪个子页面创建的cookie，所有的页面都可以访问到了，

```javascript
document.cookie = "name=Darren;path=/";
```

## cookie域概念

路径能解决在同一个域下访问 cookie 的问题，咱们接着说 cookie 实现同域之间访问的问题。语法如下，

```javascript
document.cookie = "name=value;path=path;domain=domain";
```

domain就是设置的cookie域的值。

例如"www.qq.com"与"sports.qq.com"公用一个关联的域名"qq.com"，我们如果想让 "sports.qq.com"下的cookie被 "www.qq.com" 访问，我们就需要用到cookie的domain属性，并且需要把path属性设置为 "/"。

```javascript
document.cookie = "username=Darren;path=/;domain=qq.com"
```

**注：一定的是同域之间的访问，不能把domain的值设置成非主域的域名。**


## cookie安全性

通常cookie信息都是使用HTTP连接传递数据，这种传递方式很容易被查看，所以cookie存储的信息容易被窃取。假如cookie中所传递的内容比较重要，那么就要求使用加密的数据传输。所以cookie的这个属性的名称是`secure`，默认的值为空。如果一个cookie的属性为secure，那么它与服务器之间就通过HTTPS或者其它安全协议传递数据。语法如下：

```javascript
document.cookie = "username=Darren;secure";
```

把cookie设置为secure，**只保证cookie与服务器之间的数据传输过程加密**，而保存在本地的cookie文件并不加密。如果想让本地cookie也加密，得自己加密数据。注：就算设置了`secure`属性也并不代表他人不能看到你机器本地保存的cookie信息，所以说到底，别把重要信息放cookie就对了。


## cookie编码细节

原本来想在常见问题那段介绍cookie编码的知识，因为如果对这个不了解的话编码问题确实是一个坑，所以还是详细说说。在输入cookie信息时不能包含空格，分号，逗号等特殊符号，而在一般情况下，cookie信息的存储都是采用未编码的方式。所以，在设置cookie信息以前要先使用`escape()`函数将cookie值信息进行编码，在获取到 cookie值得时候再使用`unescape()`函数把值进行转换回来。如设置cookie时：

```javascript
document.cookie = name + "=" + escape (value);
```

再看看基础用法时提到过的getCookie()内的一句：

```javascript
return unescape(document.cookie.substring(c_start,c_end));
```

这样就不用担心因为在cookie值中出现了特殊符号而导致cookie信息出错了。

End. All rights reserved `@gejiawen`.
