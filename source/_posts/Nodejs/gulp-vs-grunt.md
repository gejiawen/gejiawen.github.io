title: gulp vs. grunt
date: 2015-01-09 13:17:58
categories: [Nodejs]
tags: [gulp, grunt, nodejs, nodejs-工具探索]
---

[Gulp](http://gulpjs.com/)和[Grunt](http://gruntjs.com/)都是用于前端开发的构建工具（或者说它们是任务运行管理器更加合适一点）。其中Grunt的出现要比Gulp早不少时间，不过Gulp后来居上隐隐有取而代之的态势。

下面我将会对grunt及gulp先做一个简单的用法介绍，不过会着重介绍下gulp，最后再简单的将两者做一个对比。

# grunt简介

[gruntjs中文社区](http://www.gruntjs.net/)介绍grunt是这么说的，**JavaScript世界的构建工具**。

其实构建工具基本上就是让一些重复的任务自动化。我们先来上一个简单使用grunt的例子来大概了解下grunt是怎么用的。

首先是安装，grunt有一个对应的grunt-cli，我们需要全局安装这个cli，

```shell
~ npm install -g grunt-cli
```

然后切换到项目目录，安装grunt及用到的插件，因为是仅作示例，我这里就使用了一个插件。

```shell
~ cd <YOUR_PROJECT_DIR>
~ npm install gulp grunt-contrib-uglify --save-dev
```

然后创建`Gruntfile.js`文件，其内容如下，

```javascript
module.exports = function(grunt) {

    // Project configuration.
    grunt.initConfig({
        pkg: grunt.file.readJSON('package.json'),
        uglify: {
            options: {
                banner: '/*! <%= pkg.name %> <%= grunt.template.today("yyyy-mm-dd") %> */\n'
            },
            build: {
                src: 'src/*.js',
                dest: 'build/all.min.js'
            }
        }
    });

    // 加载包含 "uglify" 任务的插件。
    grunt.loadNpmTasks('grunt-contrib-uglify');

    // 默认被执行的任务列表。
    grunt.registerTask('default', ['uglify']);

};
```

从Gruntfile.js文件中可以看出文件内容主要分为两块，第一块是相关task的配置，第二块是加载任务。

本示例中，我仅仅配置了一个叫做**`uglify`**的任务，用于压缩代码文件。具体的策略是：压缩src目录下的所有js文件，然后将压缩好的文件合并在一起输入到dest目录下的all.min.js文件中。（这里可以看出，这个uglify插件除了压缩之外还做了合并的事。）

结果如下，

```shell
Running "uglify:build" (uglify) task
>> 1 file created.

Done, without errors.
```

更多的内容，请参考grunt的[官方文档](http://gruntjs.com/getting-started)。


# gulp简介

gulp是一款开源的**基于流的构建工具**，托管在github上，地址在[这里](https://github.com/gulpjs/gulp)，目前已经有10000+个Star，相比[grunt](https://github.com/gruntjs/grunt)的近9000个Star，后来居上，发展势头很迅猛。

下面我们通过一个简单的例子来领略下gulp的风采。

首先是安装，下面的两种方式可以根据你的喜好和情况自由选择，

```shell
~ npm install -g gulp
```

```shell
~ cd <YOUR_PROJECT_DIR>
~ npm install gulp --save-dev
```

然后是在项目的根目录下创建`Gulpfile.js`文件，

```javascript
var gulp = require('gulp');
    uglify = require('gulp-uglify'),
    concat = require('gulp-concat');

gulp.task('default', function () {
    return gulp.src('./src/*.js')
        .pipe(uglify())
        .pipe(concat('all.min.js'))
        .pipe(gulp.dest('./build'));
});
```

最后就可以运行了，

```shell
~ gulp
```
结果如下，

```shell
[14:14:13] Using gulpfile D:\WebstormProjects\hello-gulp\gulpfile.js
[14:14:13] Starting 'default'...
[14:14:13] Finished 'default' after 37 ms
```

啊哈，是不是很简单轻量呢？

## 轻量简洁的API

gulp的[API文档](https://github.com/gulpjs/gulp/blob/master/docs/API.md)中有对其API的详细描述。总的来说，gulp原生提供了四个api接口，

1. `gulp.src(globs[, options])` 指明源文件，所有gulp的task都是从读取源文件开始
2. `gulp.dest(path)` 指明任务处理的目标输出路径
3. `gulp.task(name[, deps], fn)` 注册任务
4. `gulp.watch` 监视文件改动并运行相关逻辑或者任务

## 稍微完整一些的示例

我们使用gulp来完成以下三个构建任务：1.语法检查（gulp-jshint），2.合并文件（gulp-concat），3.压缩代码（gulp-uglify）。完整的代码如下，

```javascript
var gulp = require('gulp');
var jshint = require('gulp-jshint');
var concat = require('gulp-concat');
var uglify = require('gulp-uglify');
var rename = require('gulp-rename');

// 语法检查
gulp.task('jshint', function () {
    return gulp.src('src/*.js')
        .pipe(jshint())
        .pipe(jshint.reporter('default'));
});

// 合并文件之后压缩代码
gulp.task('minify', function () {
     return gulp.src('src/*.js')
        .pipe(concat('all.js'))
        .pipe(gulp.dest('dist'))
        .pipe(uglify())
        .pipe(rename('all.min.js'))
        .pipe(gulp.dest('dist'));
});

// 监视文件的变化
gulp.task('watch', function () {
    gulp.watch('src/*.js', ['jshint', 'minify']);
});

// 注册缺省任务
gulp.task('default', ['jshint', 'minify', 'watch']);
```

除了`watch`任务外，基本上所有的任务单元都是这么样的一个模式，

```javascript
gulp.task('task-name', function () {
    var stream = gulp.src('...') // 流开始
        .pipe(...)
        .pipe(...)
        // 任务的最后一步
        .pipe(...);

    return stream;
});
```

所以，为什么说gulp是一种基于**流**的构建工具，从这里可以看出，每个任务从`src`读取源文件生成流开始，中间通过`pipe`传递流，所有的任务步骤都是在流上做一些操作，最后将流通过`dest`输出到目标上。

# gulp和grunt的差异对比

在github上有相当一些项目是用grunt作为构建工具的，观察这些项目的Gruntfile.js文件，可以很容易发现大部分的Gruntfile.js都非常臃肿，有一大段一大段的嵌套配置结构，令人看起来很烦躁。这些冗余的`Gruntfile.js`文件其实维护起来并不是那么轻量的，主要体现在下面几个方面，

1. 配置和运行分离
2. 大部分插件的职责不单一，做了不仅一件事
3. 配置项过多，做的事情越多，配置增长的就越快，且越不可控制
4. 任务执行中会产生一些临时文件，较频繁的IO操作导致性能滞后

我们再来看看gulp中是如何解决这些问题的，

1. gulp遵循**code over configuration**的原则，直接就在调用的地方配置
2. gulp的插件严格遵循单一职责原则，一个插件仅作一件事，一个gulp一般只有20多行代码就搞定
3. gulp基于流的思想，并没有过多的配置，任务更多的像是在写代码而不是定义配置
4. 这个是grunt的致命伤，gulp的流式构建改变了底层的流程控制，不再频繁的去进行IO操作了


# 参考列表

- [Stream handbook](https://github.com/substack/stream-handbook)
- [Gulp slides](http://slides.com/contra/gulp)
- [Gulp挑战Grunt，背后的哲学](http://www.jianshu.com/p/3779f708f5d7/)
- [“流式”前端构建工具——gulp.js 简介](http://segmentfault.com/blog/nightire/1190000000435599)


End. All rights reserved `@gejiawen`.
