title: jsdoc简明指南
date: 2015-03-23 16:39:39
categories: [Javascript]
tags: [jsdoc]
---

# 为何要使用jsdoc

[JSDoc](http://usejsdoc.org/index.html)是一个根据js文件中的注释信息自动生成相关文档的工具。生成的文档都是html文件，你可以将其放到web server上，其内建了各个API条目之间的跳转，非常方便。和JavaDoc和PhpDoc类似。

详细的用法可以去jsdoc的官网查阅相关文档，官网的文档做的也比较详细，并配有相关示例，应该能够比较容易的上手。

写这篇文章的目的是这样的。我读过不少前端大神在github上的源码，窃以为[TJ大神](https://github.com/tj)的代码质量最高。那么他的代码究竟质量高在哪里呢？这里我就说两点，第一是**条理清晰**，第二是**注释完备**。个人觉得，想去看源码的应该都不会是新手，如果代码具备了条理清晰和注释完备这两点，那么基本上源码阅读应该不是一个痛苦的事情。

当然，TJ大神的代码质量高可能不止是我说的这两点，比如他写的东西一般都会有完备的单元测试，等等。

所以，一个想致力提升自身水平的前端人员，都应该尽力让自己产出的代码走高质路线，特别是在团队合作中。抛开其他不谈，代码的注释其实是非常powerful的一点。

## tj的示例代码

下面先来一段[commander.js](https://github.com/tj/commander.js)中的代码段，来看下具有完备注释的js代码究竟有怎样的魅力，

```javascript
/**
 * Add command `name`.
 *
 * The `.action()` callback is invoked when the
 * command `name` is specified via __ARGV__,
 * and the remaining arguments are applied to the
 * function for access.
 *
 * When the `name` is "*" an un-matched command
 * will be passed as the first arg, followed by
 * the rest of __ARGV__ remaining.
 *
 * Examples:
 *
 *      program
 *        .version('0.0.1')
 *        .option('-C, --chdir <path>', 'change the working directory')
 *        .option('-c, --config <path>', 'set config path. defaults to ./deploy.conf')
 *        .option('-T, --no-tests', 'ignore test hook')
 *
 *      program
 *        .command('setup')
 *        .description('run remote setup commands')
 *        .action(function() {
 *          console.log('setup');
 *        });
 *
 *      program
 *        .command('exec <cmd>')
 *        .description('run the given remote command')
 *        .action(function(cmd) {
 *          console.log('exec "%s"', cmd);
 *        });
 *
 *      program
 *        .command('teardown <dir> [otherDirs...]')
 *        .description('run teardown commands')
 *        .action(function(dir, otherDirs) {
 *          console.log('dir "%s"', dir);
 *          if (otherDirs) {
 *            otherDirs.forEach(function (oDir) {
 *              console.log('dir "%s"', oDir);
 *            });
 *          }
 *        });
 *
 *      program
 *        .command('*')
 *        .description('deploy the given env')
 *        .action(function(env) {
 *          console.log('deploying "%s"', env);
 *        });
 *
 *      program.parse(process.argv);
 *
 * @param {String} name
 * @param {String} [desc] for git-style sub-commands
 * @return {Command} the new command
 * @api public
 */

Command.prototype.command = function(name, desc) {
  var args = name.split(/ +/);
  var cmd = new Command(args.shift());

  if (desc) {
    cmd.description(desc);
    this.executables = true;
    this._execs[cmd._name] = true;
  }

  this.commands.push(cmd);
  cmd.parseExpectedArgs(args);
  cmd.parent = this;

  if (desc) return this;
  return cmd;
};
```

看到了没有，代码就20来行，但是代码前的注释非常详尽，包括基本描述、使用示例、参数说明等等。

当然，我们在实际写代码时并没有必要对每一块的代码的注释都写得如此详尽。具体如何取舍，那就得看情况了。

## 使用jsdoc的真正目的

除此之外，个人觉得完备的代码注释在多人团队合作中还有另外一个好处。在团队开发中，自己需要去看别人的代码实现是一件比较经常的事情，但是很多时候你去看别人的代码的目的其实仅仅是为了知道如何使用别人写的代码，别人代码的具体实现细节你可能是不太关心的。那么此时如果有这样的完备注释，那么基本上你仅仅需要看下注释就行了，不太需要去研究别人的实现细节了（除非你看别人的代码是review code），大大减少了生产者和消费者之间的沟通成本。

个人觉得（最起码我自己打算这么做），我们使用jsdoc可能有时候并不是要真正的生成一些html文档页面（当然这个也是可以做的），而是在自己以后或者其他人阅读代码时提供帮助。


# jsdoc常用标签的说明

## 开始

```javascript
/** 你的注释 */
```

如上面的代码段，jsdoc会从js文件中以`/**`开头的注释文本抽取需要的信息。所以`/*`、`/***`等等之类开头的注释jsdoc都是不承认的。

## 常用标签

jsdoc中，以`@`开头的部分字段（关键字）都有特定含义，比如下面这段代码，

```javascript
/**
 * @param {string} somebody - Somebody's name
 */
function sayHello(somebody) {
    alert('Hello ' + somebody);
}
```

- **`@param`**，用于申明参数，参数名为`somebody`
- **`{string}`**，表示参数的类型
- **`- xxxxxx`**，对应参数的描述

好了，基本上jsdoc中所有的`@xxx`标签都遵循这一标准格式。下面描述一些常用的标签

方法参数的相关声明。Usage，

**`@param {类型} 参数名 - 参数描述`**

- 如果参数名被`[参数名]`标识，则表示此参数为可选
- `参数名=默认值`，表示参数的默认值
- `{类型1|类型2}`，表示此参数可接受多个数据类型

方法返回值的相关声明。Usage，

**`@returns {类型} 返回值描述`**

额外信息的描述标签。Usage，

**`@author`**，作者描述
**`@todo`**，待做事项描述
**`@file`**，文件描述，
**`@deprecated`**，不推荐使用（废弃）标识
**`@description(@desc)`**，描述标识

其他更多标签的说明请参阅[官方文档](http://usejsdoc.org/index.html)。

# jsdoc用于构建系统中

jsdoc还可以用在前端工程构建系统中，这样可以在项目release时一并生成代码的API文档。

在目前有两种比较常用的构建系统中都有相应的插件（[gulp-jsdoc](https://www.npmjs.com/package/gulp-jsdoc)、[grunt-doc](https://www.npmjs.com/package/grunt-jsdoc)）进行支持。两者的使用都非常简单，

gulp中，

```javascript
var jsdoc = require('gulp-jsdoc');

gulp.src('./src/*.js')
    .pipe(jsdoc('./doc'));
```

grunt中，

```javascript
grunt.initConfig({
    jsdoc : {
        dist : {
            src: ['src/*.js'],
            options: {
                destination: 'doc'
            }
        }
    }
});
```

# 自己的示例

最后贴一个上周写的[小玩儿意](https://github.com/gejiawen/bullhead/blob/master/index.js)，很简单的一个js文件，主要是观摩一下使用jsdoc给js代码添加注释的那种美感。



End. All rights reserved `@gejiawen`.
