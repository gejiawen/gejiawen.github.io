postid: "small-talk-about-fe-spec"
title: AMD/CMD与前端规范
date: 2014-07-18 17:10:07
categories: [大前端]
tags: [fe, javascript, 前端模块化]
---

# AMD与CMD

- AMD规范：[异步模块定义](https://github.com/amdjs/amdjs-api/wiki/AMD)
- CMD规范：[通用模块定义](https://github.com/seajs/seajs/issues/242)

这些规范的目的都是为了**JavaScript的模块化开发**，特别是在浏览器端的。目前这些规范的实现都能达成浏览器端模块化开发的目的。

**区别**，[这里](https://github.com/seajs/seajs/issues/277)有更详细的说明，
- 对于依赖的模块，AMD 是提前执行，CMD 是延迟执行。
- CMD 推崇依赖就近，AMD 推崇依赖前置。


# 个人理解

## CMD

CMD推崇依赖就近，可以把依赖写进你的代码中的任意一行， 如

```javascript
define(function(require, exports, module) {
    var a = require('./a');
    a.doSomething();
    var b = require('./b');
    b.doSomething();
})；
```
## AMD

AMD是依赖前置的，换句话说，在解析和执行当前模块之前，模块作者必须指明当前模块所依赖的模块。如

```javascript
define(['./a','./b'],function(a,b) {
     a.doSomething()
     b.doSomething()
});
```

代码一旦运行到此处，能立即知晓依赖。而无需遍历整个函数体找到它的依赖，因此性能有所提升，缺点就是开发者必须显式得指明依赖——这会使得开发工作量变大。比如，当你写到函数体内部几百上千行的时候，忽然发现需要增加一个依赖，你不得不回到函数顶端来将这个依赖添加进数组。

## 硬依赖与软依赖

依赖分为**硬依赖**和**软依赖**， 如，

```javascript
if (status) {
    a.doSomething();
}
```

此时，在`if`体中，可能依赖`a`，也可能不依赖`a`，这种可以称之为**软依赖**。

### 软依赖

对于软依赖的处理，我推荐依赖前置+回调函数的实现形式。如，

```javascript
if(status){
    async(['a'],function(a){
        a.doSomething()
    });
}
```

### 硬依赖

对于硬依赖（强依赖），如果要性能优先，则考虑参照依赖前置的思想设计你的模块加载器，我个人也更推崇这个方案一些；如果考虑开发成本优先，则考虑按照依赖就近的思想设计你的模块加载器。对于弱依赖，只需要将弱依赖的部分改写到回调函数内即可。


# CommonJS规范

## CommonJS介绍

CommonJS是服务端模块的规范，Node.js采用了这个规范。

根据CommonJS规范，一个单独的文件就是一个模块。加载模块使用`require`方法，该方法读取一个文件并执行，最后返回文件内部的`exports`对象。如下代码：

```javascript
console.log("evaluating example.js");

var invisible = function () {
    console.log("invisible");
};

exports.message = "hi";

exports.say = function () {
    console.log(message);
};
```

使用require方法，加载example.js。

```javascript
var example = require('./example.js');
```

这时，变量`example`就对应模块中的`exports`对象，于是就可以通过这个变量，使用模块提供的各个方法。

```javascript
{
    message: "hi",
    say: [Function]
}
```

require方法默认读取js文件，所以可以省略js后缀名。

```javascript
var example = require('./example');
```

js文件名前面需要加上路径，可以是相对路径（**相对于使用require方法的文件**），也可以是绝对路径。如果省略路径，node.js会认为，你要加载一个核心模块，或者已经安装在本地 node_modules 目录中的模块。如果加载的是一个目录，node.js会首先寻找该目录中的`package.json`文件，加载该文件`main`属性提到的模块，否则就寻找该目录下的`index.js`文件。


## 最简单的例子

下面的例子是使用一行语句，定义一个最简单的模块。

```javascript
// addition.js
exports.do = function(a, b) {
    return a + b
};
```

上面的语句定义了一个加法模块，做法就是在`exports`对象上定义一个`do`方法，那就是供外部调用的方法。使用的时候，只要用`require`函数调用即可。

```javascript
var add = require('./addition');
add.do(1,2) // 3
```

## 稍微复杂的例子

再看一个复杂一点的例子。

```javascript
// foobar.js
function foobar(){
    this.foo = function(){
        console.log('Hello foo');
    };
    this.bar = function(){
        console.log('Hello bar');
    };
}
exports.foobar = foobar;
```

调用该模块的方法如下：

```javascript
var foobar = require('./foobar').foobar,
    test = new foobar();
test.bar(); // 'Hello bar'
```

有时，不需要`exports`返回一个对象，只需要它返回一个函数。这时，就要写成`module.exports`。

```javascript
module.exports = function () {
    console.log("hello world")
};
```


# AMD规范与CommonJS规范的兼容性

**CommonJS规范加载模块是同步的**，也就是说，只有加载完成，才能执行后面的操作。**AMD规范则是非同步加载模块，允许指定回调函数**。

由于Node.js主要用于服务器编程，模块文件一般都已经存在于本地硬盘，所以加载起来比较快，不用考虑非同步加载的方式，所以CommonJS规范比较适用。但是，如果是浏览器环境，要从服务器端加载模块，这时就必须采用非同步模式，因此浏览器端一般采用AMD规范。

AMD规范使用define方法定义模块，下面就是一个例子：

```javascript
define(['package/lib'], function(lib) {
    function foo() {
        lib.log('hello world!');
    }

    return {
        foo: foo
    };
});
```

AMD规范允许输出的模块兼容CommonJS规范，这时define方法需要写成下面这样：

```javascript
define(function( require, exports, module )
    var someModule = require( "someModule" );
    var anotherModule = require( "anotherModule" );

    someModule.doTehAwesome();
    anotherModule.doMoarAwesome();

    exports.asplode = function() {
        someModule.doTehAwesome();
        anotherModule.doMoarAwesome();
    };
});
```

