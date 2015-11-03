postid: 'what-is-amd-cmd-commonjs-umd'
title: 关于AMD,CMD,CommonJS及UMD规范
categories: [大前端]
tags: [翻译, 前端规范]
date: 2015-11-03 14:01:28

---

[点我](http://davidbcalhoun.com/2014/what-is-amd-commonjs-and-umd/)看原文。

译文开始。

Javascript的组件生态在最近几年的发展很给力，我们的可选性更加广泛了。这本是一件好事，但是当多个第三方Javascript在一起混合使用的时候，我们可能会遇到一个很尴尬的问题，那就是不是所有的组件都能在一起很愉快的玩耍的。

为了解决这个问题，两种竞争关系的前端模块规范（AMD和CommonJS）问世了。他们规定开发者们采用一种约定好的模式来写代码，以避免污染整个生态系统。

# AMD规范

AMD规范，全称"Asynchronous Module Definition"，称为**异步模块加载规范**。一般应用在浏览器端。流行的浏览器端异步加载库[RequireJS](http://www.requirejs.org)（[中文网站](http://www.requirejs.cn)）实现的就是AMD规范。

下面是使用AMD规范定义一个名为`foo`模块的方式，此模块依赖jquery，

```javascript
//    filename: foo.js
define(['jquery'], function ($) {
    //    methods
    function myFunc(){};

    //    exposed public methods
    return myFunc;
});
```

AMD讲究的是前置执行。稍微复杂的例子如下，`foo`模块有多个依赖及方法暴漏，

```javascript
//    filename: foo.js
define(['jquery', 'underscore'], function ($, _) {
    //    methods
    function a(){};    //    private because it's not returned (see below)
    function b(){};    //    public because it's returned
    function c(){};    //    public because it's returned

    //    exposed public methods
    return {
        b: b,
        c: c
    }
});
```

`define`是AMD规范用来声明模块的接口，示例中的第一个参数是一个数组，表示当前模块的依赖。第二个参数是一个回调函数，表示此模块的执行体。只有当依赖数组中的所有依赖模块都是可用的时，AMD模块加载器（比如RequireJS）才会去执行回调函数并返回此模块的暴露接口。

注意，回调函数中参数的顺序与依赖数组中的依赖顺序一致。（即：`jquery`->`$`，`underscore`->`_`）

当然，在这里我可以将回调函数的参数名称改成任何我们想用的可用变量名，这并不会对模块的声明造成任何影响。

除此之外，你不能在模块声明的外部使用`$`或者`_`，因为他们只在模块的回调函数体中才有定义。

关于AMD规定声明模块的更多内容，请参考[这里](https://github.com/amdjs/amdjs-api/wiki/AMD#using-require-and-exports)。

# CMD规范

CMD规范，全称"Common Module Definition"，称为**通用模块加载规范**。一般也是用在浏览器端。浏览器端异步加载库[Sea.js](http://seajs.org/docs/)实现的就是CMD规范。

下面是使用AMD规范定义一个名为`foo`模块的方式，此模块依赖jquery，

```javascript
define(function (require, exports, module) {
    // load dependence
    var $ = require('jquery');
    
    //    methods
    function myFunc(){};

    //    exposed public methods
    return myFunc;
})
```

CMD规范倾向依赖就近，稍微复杂一点例子

```javascript
define(function (requie, exports, module) {
    // 依赖可以就近书写
    var a = require('./a');
    a.test();
    
    // ...
    // 软依赖
    if (status) {
        var b = requie('./b');
        b.test();
    }
});
```

关于AMD和CMD的区别，可参考这篇文章，[AMD/CMD与前端规范](http://gejiawen.github.io/2014/07/18/small-talk-about-fe-spec/)。

# CommonJS规范

根据CommonJS规范，一个单独的文件就是一个模块。每一个模块都是一个单独的作用域，也就是说，在一个文件定义的变量（还包括函数和类），都是私有的，对其他文件是不可见的。

如果你在Node.js平台上写过东西，你应该会比较熟悉CommonJS规范。与前面的AMD及CMD规范不一样的是，CommonJS规范一般应用于服务端（Node.js平台），而且CommonJS加载模块采用的是同步方式（这跟他适用的场景有关系）。同时，得力于[Browserify](https://github.com/substack/node-browserify)这样的第三方工具，我们可以在浏览器端使用采用CommonJS规范的js文件。

下面是使用CommonJS规范声明一个名为`foo`模块的方式，同时依赖`jquery`模块，

```javascript
//    filename: foo.js
//    dependencies
var $ = require('jquery');

//    methods
function myFunc(){};

//    exposed public method (single)
module.exports = myFunc;
```

稍微复杂一点的示例如下，拥有多个依赖以及抛出多个接口，

```javascript
//    filename: foo.js
var $ = require('jquery');
var _ = require('underscore');

//    methods
function a(){};    //    private because it's omitted from module.exports (see below)
function b(){};    //    public because it's defined in module.exports
function c(){};    //    public because it's defined in module.exports

//    exposed public methods
module.exports = {
    b: b,
    c: c
};
```

# UMD规范

因为AMD，CommonJS规范是两种不一致的规范，虽然他们应用的场景也不太一致，但是人们仍然是期望有一种统一的规范来支持这两种规范。于是，UMD（Universal Module Definition，称之为**通用模块规范**）规范诞生了。

客观来说，这个UMD规范看起来的确没有AMD和CommonJS规范简约。但是它支持AMD和CommonJS规范，同时还支持古老的全局模块模式。

我们来看个示例，

```javascript
(function (root, factory) {
    if (typeof define === 'function' && define.amd) {
        // AMD
        define(['jquery'], factory);
    } else if (typeof exports === 'object') {
        // Node, CommonJS-like
        module.exports = factory(require('jquery'));
    } else {
        // Browser globals (root is window)
        root.returnExports = factory(root.jQuery);
    }
}(this, function ($) {
    //    methods
    function myFunc(){};

    //    exposed public method
    return myFunc;
}));
```

> 个人觉得UMD规范更像一个语法糖。应用UMD规范的js文件其实就是一个立即执行函数。函数有两个参数，第一个参数是当前运行时环境，第二个参数是模块的定义体。在执行UMD规范时，会优先判断是当前环境是否支持AMD环境，然后再检验是否支持CommonJS环境，否则认为当前环境为浏览器环境（`window`）。当然具体的判断顺序其实是可以调换的。

下面是一个更加复杂的示例，

```javascript
(function (root, factory) {
    if (typeof define === 'function' && define.amd) {
        // AMD
        define(['jquery', 'underscore'], factory);
    } else if (typeof exports === 'object') {
        // Node, CommonJS-like
        module.exports = factory(require('jquery'), require('underscore'));
    } else {
        // Browser globals (root is window)
        root.returnExports = factory(root.jQuery, root._);
    }
}(this, function ($, _) {
    //    methods
    function a(){};    //    private because it's not returned (see below)
    function b(){};    //    public because it's returned
    function c(){};    //    public because it's returned

    //    exposed public methods
    return {
        b: b,
        c: c
    }
}));
```

**Tips**: 如果你写了一个小工具库，你想让它及支持AMD规范，又想让他支持CommonJS规范，那么采用UMD规范对你的代码进行包装吧，就像[这样](https://github.com/gejiawen/bullhead/blob/master/index.js)。







