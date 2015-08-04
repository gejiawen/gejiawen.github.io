title: JavaScript轻量级框架系列（1）--超炫3D效果的幻灯片框架Impress.js
date: 2014-09-23 10:30:21
categories: [Vendors]
tags: [JavaScript轻量级框架系列, javascript]
---

本文是[JavaScript轻量级框架系列](http://gejiawen.github.io/2014/11/26/%E7%B3%BB%E5%88%97/JavaScript%E8%BD%BB%E9%87%8F%E7%BA%A7%E6%A1%86%E6%9E%B6%E7%B3%BB%E5%88%97/)的第**一**篇文章。

**Impress.js**是基于*webkit*内核浏览器（Chrome、Safari等）开发，而在其他基于非webkit引擎但支持*CSS3 3D*的浏览器也能正常运行。

- Impress.js源码已经发布在Github上，地址：[https://github.com/bartaz/impress.js](https://github.com/bartaz/impress.js/)
- 官方demo地址：[http://bartaz.github.com/impress.js](http://bartaz.github.com/impress.js)



# 配置

- html5页面结构先准备就绪
- 创建一个`id="impress"`的*wrapper*，可以直接是一个`div`，当然其他标签也是可以的
- 在`body`标签结束前引入`impress.js`文件并且调用
- `class="impress-not-supported"`是当浏览器不支持时显示给用户的提示信息，降级处理

```javascript
<!doctype html>
<html>
<head>
    <title>darren - Impress demo</title>
    <meta charset="utf-8" />
    <link href="http://bartaz.github.com/impress.js/css/impress-demo.css" rel="stylesheet" />
</head>
<body>
    <div class="impress-not-supported"></div>
    <div id="impress"></div>
    <script src="http://bartaz.github.com/impress.js/js/impress.js"></script>
    <script>
        impress().init();
    </script>
</body>
</html>
```

在*wrapper*内创建一个幻灯片只需要新建一个`class="step"`的`<div>`即可。
`<div>`的`id`可有可无，当有`id`时`url`中的`hash`变化是随着`id`走；反之就是`step-[num]`。

# 数据属性

- data-x    幻灯片的x坐标
- data-y    幻灯片的y坐标
- data-scale    通过指定一个值来进行缩放，data-scale为5则将会在你幻灯片原始尺寸基础上放大5倍
- data-rotate   通过一个数字度数来确定旋转你的幻灯片
- data-rotate-x     为3D用，这个数字度数是它应该相对 x轴 旋转多少度（前倾、后仰）
- data-rotate-y     为3D用，这个数字度数是它应该相对 y轴 旋转多少度（左摆、右摆）
- data-rotate-z     为3D用，这个数字度数是它应该相对 z轴 旋转多少度


# 创建

从一个出师的幻灯片开始，这个幻灯片已将它的`data-x`和`data-y`数据属性设置为**0**，所以会出现在页面的中间。

```html
<div class="step" data-x="0" data-y="0">
    This is slide1
</div>
```

第二个幻灯片的`data-x`的值为**500**，`data-y`的值为**0**，活动的时候它将会向左平移（滑动）**500px**的地方。

```html
<div class="step" data-x="500" data-y="0">
    This is slide2
</div>
```

第三张幻灯片其`data-x`值不变，`data-y`为**-400**，这将会是从顶部**400px**处滑入屏幕。

```html
<div class="step" data-x="500" data-y="-400">
    This is slide3
</div>
```

第四张幻灯片来个新花样，使用`data-scale`的值控制其缩放大小。`data-scale="0.5"`表示着它应该是一半的尺寸，当它变成活动的演示时将通过必需的倍数调节所有幻灯片的缩放尺寸，从这一步绚丽开始起步。

```html
<div class="step" data-x="500" data-y="-800" data-scale="0.5">
    This is slide 4
</div>
```

第五张幻灯片旋转属性允许你旋转一个幻灯片到当前视图，幻灯片5被设置旋转90度，视觉效果略屌哈。

```html
<div class="step" data-x="0" data-y="-800" data-rotate="90">
    This is slide 5
</div>
```

第六张幻灯片开始*3D style*，可为每个维度的轴指定旋转属性(x,y,z)。x轴是横轴，意思是你可使事物倾斜(正值)或向后(负值)，y轴是竖轴，所以你可使事物向左摇摆(负值)或向右(正值)，z轴是纵轴，这将是旋转的东西向上（负值）和向下（正值）。

```html
<div class="step" data-x="-1200" data-y="0" data-rotate-x="30" data-rotate-y="-30" data-rotate-z="90" data-scale="4">
    This is slide 6
</div>
```

# 全局预览

个人超赞这个视觉体验，把所有的幻灯片都平行的展示，排列的合理会非常帅气，使用方式就是在*幻灯片6*后面插入一段`html`。

```html
<div id="overview" class="step" data-x="-200" data-y="-500" data-scale="3"></div>
```

随着你幻灯片位置的不同所以全局预览的值也会不一样，拿着结尾处的demo一点一点调整找感觉，希望你会喜欢！

完成后请记住它，用它做的不只局限于此，唯一的限制是你的创造力！


# 总结

正因为我们是前端，所以用前端技术做做各种尝试没什么不好，`impress.js`更可以让我们的演示文稿更有新意，所以简单了解下绝对是值得的，**学习是最好的投资**。

优点，

- 个人非常喜欢overview的功能
- 因为HTML+CSS都需要自己完成，位置和效果都得自己经手，视觉效果都由自己掌控
- 在我用过的同类产品中视觉效果最绚，CSS3+3D效果，直接给观众看晕:)

缺点，

- impress在视觉表现上确实非常强大，比起同样做演示文稿的`html5slides`和`deck.js`，`impress.js`的复杂度上高了不少，而且如果想把演示文稿排版的好看可能需要花掉大量的时间。
    - 如果嫌`impress.js`麻烦的朋友可以去看看`html5slides`和`deck.js`的资料，视觉效果会稍差一些，不过上手会简单不少
- 不要把3D和旋转用得太花哨、太绚，看的人会晕，恰当就好。


End. All rights reserved `@gejiawen`.
