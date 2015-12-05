postid: 'gulp-reminder-1-api'
title: gulp备忘录（一）：API指南
categories: [Nodejs]
tags: [gulp, nodejs, 漫游NodeJS系列]
date: 2015-12-05 22:58:55

---

# 前言

自从[gulp](http://gulpjs.com/)面世以后，我就抛弃了[Grunt](http://gruntjs.com/)加入了gulp的阵营。从易用性以及性能方面来说，无疑gulp更具优势。具体的可以看我之前的一篇文章[gulp vs. grunt](http://gejiawen.github.io/2015/01/09/gulp-vs-grunt/)。

可以说，我很早就开始使用gulp了，那为何会有这几篇gulp的相关文章呢？是因为发现我在使用gulp的时候，仍然处于一个**仅仅会用**的层面上，当需要使用一些插件时，往往是在[npm](https://www.npmjs.com/)找到对应的插件，然后照着example抄一遍，对gulp的有些特性还不是很清楚。所以我觉得很有必要写一些东西来深度学习一下gulp。

# 介绍

首先来介绍一下gulp。

gulp是一款前端构建工具，它将构建操作抽象成一系列的任务。做的事情与grunt是一样的。不过它信奉的原则是**代码优于配置**，基于Node.js中的stream来操作文件。

## 安装

我们可以通过以下方式来安装gulp，

```bash
$ npm install -g gulp
```

安装成功之后你就可以直接在任何路径上使用`gulp`命令了。不过在实际项目中使用gulp时，往往需要安装一个local版本的gulp，

```bash
$ npm install gulp --save-dev
```

采用`--save-dev`将gulp写进`package.json`的`devDependencies`依赖中。

至于为何全局安装gulp之后还需要在项目中安装gulp，可以参见stackoverflow上的问题[why-do-we-need-to-install-gulp-globally-and-locally](http://stackoverflow.com/questions/22115400/why-do-we-need-to-install-gulp-globally-and-locally)。基本上这么做的原因是为了灵活的去使用gulp，如果实在不能理解的话也不必去纠结。你大可以不用全局安装gulp，仅在项目中安装。

# 开始使用

在项目中添加gulp依赖并安装完毕之后，我们需要在项目的根目录中创建一个名叫`gulpfile.js`的文件。我们将在这个gulpfile.js文件中完成所有构建任务的设计。

一般地，我们gulpfile.js文件内容如下，

```javascript
var gulp = require('gulp');

gulp.task('default', function () {
    console.log('default finished.');
});
```

然后我们直接在命令行中输入`gulp`就可以开始构件任务，

```bash
[00:54:32] Using gulpfile ~/code/test/test-gulp/gulpfile.js
[00:54:32] Starting 'default'...
default finished.
[00:54:32] Finished 'default' after 98 μs
```

在本例中，我们在default任务中啥事也没干，仅仅输出一条提示信息。

基本上gulp的使用就是这样的模式。


# APIs说明

gulp的API非常简洁，常用的API就4个，他们是`gulp.task`，`gulp.src`，`gulp.dest`，`gulp.watch`。

## `gulp.task`

`gulp.task`用于定义任务。其内部使用的是一个名为[Orchestrator](https://github.com/orchestrator/orchestrator)任务调度库。它的用法很简单，

```javascript
gulp.task(name[, deps], fn)
```

| 参数 | 类型 | 是否可选 | 说明 |
| :--- | :--- | :---  | :--- |
| `name` | **String** | **必须** | 任务的名称 |
| `deps` | **Array** | 可选 | 是一个数组，表示当前任务所依赖的其他任务 |
| `fn` | **Function** | **必须** | 任务具体要执行的操作 |

下面是一个例子，

```javascript
gulp.task('mytask', ['one', 'two', 'three'], function () {
    doSomething();
});

gulp.task('mytask2', function() {
    doSomething();
});

gulp.task('mytask3', ['one', 'two']);
```

如示例所示，当某一个任务没有依赖任务时，可以省略`deps`参数。当一个任务有依赖任务时，也是可以省略`fn`参数的。

### 控制依赖顺序

当`gulp.task`定义任务并有依赖任务时，gulp默认将以最大的并发数去同时执行这些依赖任务。所以gulp默认是不保证依赖任务之间的异步操作等待的。

若有下面这样一个示例，

```javascript
gulp.task('one', function() {
    setTimeout(function() {
        console.log('one is done.');
    }, 2000);
});

gulp.task('two', ['one'], function() {
    console.log('two is done.');
});

gulp.task('default', ['one', 'two']);
```

如果我们的`one`，`two`这两个任务彼此之间有顺序依赖的话（示例中采用setTimeout来模拟异步任务），即`two`要在`one`的异步操作完毕之后执行。如果我们直接执行`gulp`，得到结果如下，

```bash
[01:25:08] Using gulpfile ~/code/test/test-gulp/gulpfile.js
[01:25:08] Starting 'one'...
[01:25:08] Finished 'one' after 578 μs
[01:25:08] Starting 'two'...
[01:25:08] two is done.
[01:25:08] Finished 'two' after 2.33 ms
[01:25:08] Starting 'default'...
[01:25:08] Finished 'default' after 35 μs
[01:25:10] one is done.
```

可以看到，任务`one`和任务`two`几乎是在同一时刻就同时执行的。但是在*01:25:08*任务`two`就已经完成了，而任务`one`在*01:25:10*才完成。所以这并没有达到我们的要求。那么，我们该如何做呢？

通常，我们会有三种方式来达到这一目的。

第一种方式：在异步操作完成后执行一个回调函数来通知gulp这个异步任务已经完成，这个回调函数就是任务函数的第一个参数。

下面我们来改写一下任务`one`，

```javascript
gulp.task('one', function(callback) {
    setTimeout(function() {
        console.log('one is done.');
        callback();
    }, 2000);
});
```

此时我们再执行`gulp`其结果如下，

```bash
[01:40:02] Using gulpfile ~/code/test/test-gulp/gulpfile.js
[01:40:02] Starting 'one'...
[01:40:04] one is done.
[01:40:04] Finished 'one' after 2.01 s
[01:40:04] Starting 'two'...
[01:40:04] two is done.
[01:40:04] Finished 'two' after 659 μs
[01:40:04] Starting 'default'...
[01:40:04] Finished 'default' after 12 μs
```

如此我们就能保证在执行`two`之前，`one`的异步操作也执行完毕了。

下面我们介绍另外两种方式。

第二种方式：定义任务时返回一个流对象。

比如，

```javascript
gulp.task('one', function() {
    return gulp.src('client/**/*.js')
        .pipe(minify())
        .pipe(gulp.dest('build'));
});

gulp.task('two', ['one'], function() {
    console.log('two is done.');
});

gulp.task('default', ['one', 'two']);
```

第三种方式：返回一个promise对象。

比如，

```javascript
var Q = require('q');

gulp.task('one',function(cb){
    var deferred = Q.defer();
  
    setTimeout(function() {
        deferred.resolve();
    }, 5000);
  
    return deferred.promise;
});

gulp.task('two', ['one'], function(){
    console.log('two is done.');
});

gulp.task('default', ['one', 'two']);
```

**总结**，

- callback和返回promise的方式更具普适性，而返回stream的方式更加合适`gulp.src`相关的操作。
- callback方式在某些场景下可能更加灵活。比如任务中掺杂了一些业务逻辑的时候。



## `gulp.src`

前面说过gulp的工作机制是基于Node.js中的流。这里所说的流并不是Node.js原始的流对象，而是一种虚拟文件流对象（[Vinyl files](https://github.com/gulpjs/vinyl-fs)），这个虚拟文件对象中存储着原始文件的路径、文件名、内容等信息。我们不关注这个，后面再讲解写gulp插件时会再次阐述这块内容。

所以，简单来说`gulp.src`就是用来获取你需要操作的文件的。它的用法如下，

```javascript
gulp.src(globs[, options])
```

| 参数 | 类型 | 是否可选 | 说明 |
| :--- | :--- | :---  | :--- |
| `globs` | **String**, **Array** | **必选** | 文件匹配模式 |
| `options` | **Object** | 可选 | 匹配选项 |

`gulp.src`内部使用[node-glob](https://github.com/isaacs/node-glob)模块来实现文件匹配。node-glob是一种与正则表达式类似的匹配范式。

`globs`参数也可以直接是用一个文件地址，还可以是一个包含多个匹配模式的数组。而`options`参数是用来辅助匹配的，它的某些参数设置将会影响虚拟文件流的获取。

### 模式匹配

node-glob中有如下几种匹配规则，

| 匹配范式 | 用法 |
| :--- | :--- |
| `*` | 匹配0个或多个字符，但不会匹配路径分隔符，除非路径分隔符出现在末尾 |
| `**` | 匹配0个或多个目录及其子目录，需要单独出现。如果出现在末尾，也能匹配文件 |
| `?` | 匹配一个字符，不会匹配路径分隔符 |
| `[...]` | 匹配方括号中出现的字符中的任意一个，当方括号中第一个字符为^或!时，表示取反 |
| `!(p1,p2,p3)` | 匹配任何与括号中给定的任一模式都不匹配的 |
| `?(p1,p2,p3)` | 匹配括号中给定的任一模式0次或1次 |
| `+(p1,p2,p3)` | 匹配括号中给定的任一模式至少1次 |
| `*(p1,p2,p3)` | 匹配括号中给定的任一模式0次或多次 |
| `@(p1,p2,p3)` | 匹配括号中给定的任一模式1次 |
| `{p1,p2}` | 以展开模式进行匹配 |

> **备注**： `(p1,p2,p3)`中的`,`其实是`|`。如果写成`p1|p2|p3`，将会导致markdown解析错误。暂时没找到解决方案，望知道的大神告知😂

关于node-glob更多的用法，请参阅[node-glob](https://github.com/isaacs/node-glob)的相关文档。不过基本上上面的几种用法将会覆盖`gulp.src`90%以上的场景。

下面是一些更详细的示例，

- `*`能匹配`a.js`，`x.y`，`abc`，`abc/`，但不能匹配`a/b.js`
- `*.*`能匹配`a.js`，`style.css`，`a.b`，`x.y`
- `*/*/*.js`能匹配`a/b/c.js`，`x/y/z.js`，不能匹配`a/b.js`，`a/b/c/d.js`
- `**`能匹配`abc`，`a/b.js`，`a/b/c.js`，`x/y/z`，`x/y/z/a.b`，能用来匹配所有的目录和文件
- `**/*.js`能匹配`foo.js`，`a/foo.js`，`a/b/foo.js`，`a/b/c/foo.js`
- `a/**/z`能匹配`a/z`，`a/b/z`，`a/b/c/z`，`a/d/g/h/j/k/z`
- `a/**b/z`能匹配`a/b/z`，`a/sb/z`，但不能匹配`a/x/sb/z`，因为只有单`**`单独出现才能匹配多级目录
- `?.js`能匹配`a.js`，`b.js`，`c.js`
- `a??`能匹配`a.b`，`abc`，但不能匹配`ab/`，因为它不会匹配路径分隔符
- `[xyz].js`只能匹配`x.js`，`y.js`，`z.js`，不会匹配`xy.js`，`xyz.js`等，整个中括号只代表一个字符
- `[^xyz].js`能匹配`a.js`，`b.js`，`c.js`等，不能匹配`x.js`，`y.js`，`z.js`
- `a{b,c}d.js`能匹配`abd.js`，`acd.js`，不能匹配`abcd.js`

我们还可以将多个匹配模式组合成一个数组传递给`gulp.src`，比如

```javascript
gulp.src(['js/*.js', 'css/*.js', 'views/*.html']);
```

在使用这一方式时，我们还可以添加一个反模式用来在之前的匹配结果中排除某些匹配项，比如

```javascript
gulp.src(['js/*.js', '!js/jquery.js']);
```

因为反模式是在之前的匹配结果做排除的，所以反模式不能放在数组的第一个位置，比如下面这种方式是不正确用法，

```javascript
gulp.src(['!js/jquery.js', 'js/*.js']); // 反模式位于匹配模式数组的第一个位置，此时将起不到排除作用
```

### 匹配参数

我们可以通过`options`给`gulp.src`传递匹配参数。`options`除了node-glob原有的一些参数之外，还有一些额外的参数。

| 参数 | 默认值 | 说明 |
| :--- | :--- | :--- |
| `options.buffer` | `true` | gulp的虚拟文件对象默认以buffer的形式返回文件内容。若此项为`false`，则以stream的形式返回文件内容 |
| `options.read` | `true` | gulp默认是会进行读取文件内容操作的。若此项被设置为`false`则gulp则不会去读取文件 |
| `options.base` | 模式匹配中直到出现通配符之前的字符串 | 这个参数跟`gulp.dest`的最终路径息息相关 |

这些参数中，唯一值得一提就是`options.base`。具体请接着往下看。

## `gulp.dest`

`gulp.dest`简单来说就是用来写文件的。其用法如下，

```javascript
gulp.dest(path[, options])
```

| 参数 | 类型 | 是否可选 | 说明 |
| :--- | :--- | :---  | :--- |
| `path` | **String** | **必选** | 写入文件的路径 |
| `options` | **Object** | 可选 | 写入文件时的参数 |

这里唯一值得一提就是写入文件的路径。这里的写入路径是gulp结合`gulp.src`中的`options.base`参数来计算的，我们传入的`path`参数其实只是最终生成文件路径的文件夹。让我们来看一个例子，

```javascript
gulp.src('js/jquery.js')
    .pipe(gulp.dest('dist/build.js'));
```

上面的代码最终写的文件路径是**`dist/build.js/jquery.js`**，而不是`dist/build.js`。

其实，`gulp.dest`在写文件时，其步骤可以简单抽象如下，

1. 将`gulp.src`得到的文件路径分解为**base** + **matched**的形式。而base是匹配模式中**开始出现通配符之前**的路径，matched是**开始出现通配符之后**的路径。如果匹配模式中没有通配符，则matched为文件名。所以示例中的base为`js/`，matched为`jquery.js`。
2. `gulp.dest`的规则是使用写入路径参数替换base，再加上matched。即`dest-path` + `matched`。所以示例中最终的路径为`dist/build.js/jquery.js`。
3. 如果`gulp.dest`最终得到写入路径不存在，则会自动生成相应的路径。

再来看一个例子，

```javascript
// 假设要匹配的文件路径为src/public/js/jquery.js
gulp.src('src/public/**/*.js')      // base: src/public/    matched: js/jquery.js
    .pipe(gulp.dest('dist/build')); // result: dist/build/js/jquery.js

gulp.src('src/public/**/*.js', {base: 'src'}) // base: src/    matched: public/js/jquery.js
    .pipe(gulp.dest('dist'))               // dist/public/js/jquery.js
```

**注意**，

- 当手动配置`options.base`时，将导致matched也会跟着发生变化。
- 手动设置的`options.base`必须是原始base字符串的某个开始字串。比如示例中原始base串是`src/public`，那么手动设置的base只能是`src`或者`src/public`，而不能是其他的，否则将不能正确写入文件。

### 写入参数

| 参数 | 默认值 | 说明 |
| :--- | :--- | :--- |
| `options.cwd` | `process.cwd()` | 输出目录的cwd参数，只在所给的输出目录是相对路径时候有效。 |
| `options.mode` | **0777** | 八进制权限字符，用以定义所有在输出目录中所创建的目录的权限。 |

写入参数一般都不用自定义。

## `gulp.watch`

`gulp.watch`用来监视文件的变化，当文件发生变化后，我们可以利用它来执行相应的任务。其内部是基于[gaze](https://github.com/shama/gaze)实现的。

其用法如下，

```javascript
gulp.watch(glob[, opts], tasks)
```

或者

```javascript
gulp.watch(glob[, opts], fn)
```

| 参数 | 类型 | 是否可选 | 说明 |
| :--- | :--- | :---  | :--- |
| `glob` | **String**, **Array** | **必须** | 同`gulp.src` |
| `opts` | **Object** | 可选 | - |
| `tasks` | **Array** | **必须** | 为文件变化后要执行的任务，为一个数组 |
| `fn` | **Function** | **必须** | 文件变化后需要执行的回调函数，函数参数为一个对象 |


来看个例子，

```javascript
gulp.task('uglify',function(){
  //do something
});
gulp.task('reload',function(){
  //do something
});
gulp.watch('js/**/*.js', ['uglify','reload']);
```

使用`fn`的例子，

```javascript
gulp.watch('js/**/*.js', function(ev){
    console.log(ev.type); //变化类型 added为新增,deleted为删除，changed为改变 
    console.log(ev.path); //变化的文件的路径
}); 
```

# 参考列表

- [gulp](http://gulpjs.com/)
- [gulp中文网](http://www.gulpjs.com.cn/)
- [前端构建工具gulpjs的使用介绍及技巧](http://www.cnblogs.com/2050/p/4198792.html)



