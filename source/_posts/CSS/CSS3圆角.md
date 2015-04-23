title: "CSS3圆角"
date: 2015-04-23 11:52:48
categories:
tags:
---

CSS Level3中正式开始支持圆角属性，官方给出的文档在[这里](http://www.w3.org/TR/css3-background/#the-border-radius)。本篇文章将会介绍CSS3中`border-radius`的相关知识和用法。

在CSS3支持圆角之前，我们要想实现圆角效果，得用多张图片进行定位，经过一系列精细的调试才能得到想要的效果。这种方法也是无奈之举，而且很麻烦，同时它具有几个不可避免的弊端，

- 增加了页面文件及静态资源的维护成本
- 增加了额外的网络带宽消耗（因为需要加载更多的图片）

现在直接对元素设置`border-radius`属性即可达到圆角的效果。其语法如下，

```
border-radius: <length>{1-4} | % / <length>{1-4} | %;
```

即，可以设置

1. 1～4长度值
2. 一个百分比
3. 可以设置两组值，使用`／`连接

比如，我有如下的代码，

```html
<style>
    #border-radius {
        width: 200px;
        height: 200px;
        border: 1px solid;
        background-color: #0a0;
        border-radius: 20px;
    }
</style>

<div id="border-radius"></div>
```

其效果如下，

![](img-01.png)

上述的代码中，我只给`border-radius`属性设置了一个值。其实`border-radius`跟`border`属性时很相似的，它是一个复合属性，可分为*左上*、*右上*、*左下*、*右下*共四个可赋值项。前面说过，`border-radius`可以赋值1～4个值，

- 设置**1个值**时，四个方向都使用同一个值
- 设置**2个值**时，*左上*和*右下*使用第一个值，*右上*和*左下*使用第二个值
- 设置**3个值**时，*左上*使用第一个值，*右上*和*左下*使用第二个值，*右下*使用第三个值
- 设置**4个值**时，分别对应*左上*、*右上*、*左下*、*右下*的顺序进行赋值

除了直接设置1～4个长度单位之外，我们还可以设置百分比（**％**），比如

```css
#border-radius {
    width: 200px;
	height: 200px;
	background-color: #0a0;
	border: 1px solid;
	border-radius: 10% 20% 50%;
}
```

其效果如下，

![](img-02.png)

其实`border-radius`属性意为**边框半径**，它分为**水平半径**和**垂直半径**，如下图

![](img-03.png)


我们在设置`border-radius`时其实还可以使用形如下面示例的赋值方式，

```css
#border-radius {
    width: 200px;
    height: 200px;
    background-color: #0a0;
    border: 1px solid;
    border-radius: 10px 50px 15px 50px/5px 20px 10px 20px;
}
```

其效果如下，

![](img-04.png)

我们可以给水平或者垂直半径分别进行赋值。所以，我们可以给不同方向设置圆角，同时还可以分别设置其水平或者垂直半径来达到不同曲率的圆角。

当我们使用`／`设置圆角时，第一组属性为水平半径的赋值，第二组值为垂直半径赋值。且赋值的方向规律与前述一致。


除此之外，CSS3还支持对元素的某个角进行赋值，

- `border-top-left-radius`，左上角
- `border-top-right-radius`，右上角
- `border-bottom-right-radius`，右下角
- `border-bottom-left-radius`，左下角

这四个属性都可以设置1～2个值，如下

```css
.test {
    border-top-left-radius: 10px 20px;
    border-top-right-radius: 15px;
    border-bottom-right-radius: 0;
    border-bottom-left-radius: 0;
}
```

1. 若设置一个值，则表示水平半径和垂直半径相同
2. 若设置两个值，则表示分别按顺序设置水平、垂直半径



**兼容性**：虽然目前各大主流现代浏览器都支持`border-radius`属性，不过在具体的实现细节上可能会有所差别，针对不同的浏览器需要添加各自的浏览器兼容前缀。




# 参考列表

- [http://www.w3.org/TR/css3-background](http://www.w3.org/TR/css3-background/#the-border-radius)
- [CSS3圆角详解](http://www.ruanyifeng.com/blog/2010/12/detailed_explanation_of_css3_rounded_corners.html)

End! All rights reserved `@gejiawen`.
