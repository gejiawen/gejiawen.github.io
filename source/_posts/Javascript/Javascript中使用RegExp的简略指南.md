postid: 'regexp-in-javascript'
title: Javascript中使用RegExp的简略指南
date: 2015-09-17 10:42:14
categories: [Javascript]
tags: [正则表达式]

---

# 前言

RegExp是Javascript中的一个内置对象。使用它可以在javascript中进行一些正则方面的操作。本文打不打算详细的介绍RegExp的具体内容，比如子匹配、分组、惰性匹配等等这些内容的具体用法。本文仅从实用的角度来阐述regexp在javascript中的一些常用以及基本方法。关于RegExp的具体文档，可参阅[w3school](http://www.w3school.com.cn/jsref/jsref_obj_regexp.asp)或者[MDN](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/RegExp)。

# 创建

在Javascript中有两种方式来创建一个regexp。一种是使用`new RegExp(pattern[, flags])`，另一种是使用`/pattern/[flags]`。前者创建了一个regexp对象，后者是一个对象字面量。

这里有两点稍微提一下，

- 使用第一种方式时，我们可以传递一个字符串，也可以传递一个正则字面量。
- 使用字符串创建regexp对象时，注意元字符的转义。

下面是几个例子，

```javascript
var reg1 = /ab+c/i;
var reg2 = new RegExp('ab+c', 'i');
var reg3 = new RegExp(/ab+c/, 'i');

var reg4 = /\w+/;
var reg5 = new RegExp('\\w+');
```

# 方法

在Javascript中，跟regexp对象关联最为密切的一般都是String对象。支持regexp的String方法有如下几个，

| 方法 | 说明 |
| :--- | :--- |
| `search` | 检索与正则表达式相匹配的值 |
| `match` | 找到一个或多个正则表达式的匹配 |
| `replace` | 替换与正则表达式匹配的子串 |
| `split` | 把字符串分割为字符串数组 |

上面的4个方法都可以使用regexp作为参数，而且根据传入的regexp可能返回的结果会有很大的不同。

除此之外，regexp对象本身也有一些方法，他们是，

| 方法 | 说明 |
| :--- | :--- |
| `compile` | 编译正则表达式 |
| `exec` | 检索字符串中指定的值。返回找到的值，并确定其位置。 |
| `test` | 检索字符串中指定的值。返回 true 或 false。 |

关于各个方法在javascript中的具体使用示例，请继续往下看😄

# 示例

下面，我将会使用一系列的真实示例来阐述regexp以及相关方法的用法。

```javascript
// 测试`test`方法

var reg = /he/;
reg.test('hehe'); // true
reg.test('boy'); // false
reg.test('HEHE'); // false 因为regexp默认是大小写敏感的

var reg2 = /he/i;
reg2.test('HEHE'); // true
reg2.test('I am a good boy, and she is a bad girl.'); // true 只要被匹配的字符串中包含he（HE、hE、He）即可。

var reg3 = /^he/i;
reg3.test('She is a good girl'); // false 因为he不是字符串的首部。

var reg4 = /he$/i;
reg4.test('She loves he'); // true 因为he是字符串的尾部。

var reg5 = /^he$/i;
reg5.test('he'); // true 因为字符串是以h开始，以e结束，且中间没有任何其他字符。

var reg6 = /\s/; // 元字符`\s`匹配任何空白字符，包括空格、制表符、换页符等等  
reg6.test('ge jiawen'); // 字符串中有空格

var reg7 = /^[a-z]/; // 元字符`[]`匹配指定范围内的任意字符
reg6.test('good123'); // true 字符串中必须要是以字母开头

// 仅仅知道了字符串是否匹配模式还不够，我们还需要知道哪些字符匹配了模式

var reg8 = /^[a-z]+\s+\d+$/i; // 量词`+`，表示至少匹配一次
reg8.test('Ubuntu 8'); // true 因为匹配到了版本号

// 我们使用regexp的exec方法，返回一个数组，第一个元素为匹配的内容，后续的数组元素为子匹配的内容（如果有子匹配的话）

var ret = reg8.exec('Ubuntu 8');
console.log(ret[0]); // ['Ubuntu 8']

// 提取匹配内容中的数字

var reg9 = /\d+/;
reg9.exec(reg8.exec('Ubuntu 8')); // ['8']

// 要提取'Ubuntu 8'中的数字，我们还是使用子匹配

var reg10 = /^[a-z]+\s+(\d+)/i; // 一般我们使用`()`来设定一个子匹配（或者叫匹配分组）
reg10.exec('Ubuntu 8'); // ['Ubuntu 8', '8']

// 可以设定多个子匹配

var reg11 = /^[a-z]+\s+(\d+)\.(\d+)$/i;
reg11.exec('Ubuntu 14.04'); // ['Ubuntu 14.04', '14', '04']

// 使用replace，进行字符串替换

var str = 'some money';
str.replace('some', 'much'); // much money
str.replace(/\s+/, '%'); // much%money

// 使用regexp的全局匹配标志，进行多次匹配。
// 如果不进行全局匹配，那么regexp将在匹配成功一次后终止匹配。反之则匹配整个字符串。

var str2 = 'some      money'; // 包含了多个空格字符
str2.replace(/\s/, '%'); // some%   money
str2.replace(/\s/g, '%'); // some%%%%money

// split方法使用regexp。
// 在`[]`中使用`^`表示范围取反。

var str3 = 'a_bc-d=e--f';
str3.split(/[^a-z]+/g); // ["a", "bc", "d", "e", "f"]

// search使用regexp。

var str4 = 'My age is 25 age.';
str4.search(/\d+/); // 10，表示匹配字符串的开始位置。如果没有匹配到则为-1

// 使用match方法。

var str5 = 'My name is Larry. Hello everyone.';
str5.match(/[A-Z]/); // ['M'] 因为我们没有全局匹配。所以匹配成功一次就终止了
str5.match(/[A-Z]/g); // ["M", "L", "H"]
str5.match(/\b[a-z]\b/ig); // ["My", "name", "is", "Larry", "Hello", "everyone"] 匹配所有的单词
str5.match(/\b[a-z]{2}\b/ig); // ["My", "is"] 匹配所有2个字符的单词

// 捕获分组与非捕获分组。
// 捕获分组将`(abc)`中的`abc`匹配内容也放到结果数组中。而非捕获分组则不会。

/(abc){2}/.exec('abcabc'); // ["abcabc", "abc"]
/(?:abc){2}/.exec('abcabcabc') // ["abcabc"]

// 候选，也就是‘或者’的意思

/^a|c$/.test('asssc'); // true 因为将匹配开始的字符a，或者结束位置的c
/^(a|bc)$/.test('abc') // false 因为匹配的字符串必须是a或者bc

// 分组与反向引用。
// `test`,`match`等方法对包含分组的regexp进行操作时，会将分组存储起来，通过反向引用可以拿到分组。

var reg = /(A?(B?(C?)))/;
reg.test('ABC'); // 反向引用被存储在RegExp对象的静态属性$1—$9中  
console.log(RegExp.$1 + ", " + RegExp.$2 + ", " + RegExp.$3); // ABC, BC, C

var reg2 = /\d+(\D)\d+\1\d+/; // 在regexp中可以使用`\1`来反向引用匹配到的分组
'2008-1-1'.test(reg2); // true
'2008-1_1'.test(reg2); // false

var reg3 = /(\d)\s(\d)/;
'1234 5678'.replace(reg3, '$2 $1'); // '5678 1234'
 
// 正向前瞻(?=)与负向前瞻(?!)
// 正则表达式引擎在运行正向前瞻或者负向前瞻时，正则表达式引擎会留意字符串后面的部分，然后却不会真实的移动匹配指针index。

// 示例：如果我要匹配一个后面跟一个数字的单词，但是仅仅返回这个单词不包括数字。
var reg = /[a-z]+(?=\d+)/;
var str = 'I am a good2 boy.';
str.match(reg); // ['good']

// 示例：如果我要匹配一个后面不跟一个数字的单词，但是仅仅返回这个单词不包括数字。
var reg = /[a-z]+(?!\d+)\b/g;
var str = 'one1 two2 three four4';
str.match(reg); // ["three"]
```

以上代码块中的内容基本上就是javascript中使用regexp的方方面面了。掌握这些方法，在编写javascript代码时，特别是针对一些字符串进行处理时，使用regexp往往会具有很高的效率。不过，javascript中的regexp仅仅涵盖了正则表达式的基本及常见用法。要想了解更多更详细的内容，可以参阅相关专门讲述正则表达式的书籍。


# 参考列表

- [JavaScript RegExp 对象](http://www.w3school.com.cn/jsref/jsref_obj_regexp.asp)
- [MDN-RegExp](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/RegExp)
- [详述RegExp的使用](http://blog.csdn.net/qin9r3y/article/details/24982195)




