postid: '45-useful-javascript-tips-tricks-and-best-practices'
title: Javascript实用黑科技45条
categories: [Javascript]
tags: [javascript, 翻译, 黑科技]
date: 2015-09-22 20:22:00

---

本文是一篇翻译文章。原文：[45 Useful JavaScript Tips, Tricks and Best Practices](http://modernweb.com/2013/12/23/45-useful-javascript-tips-tricks-and-best-practices/)

译文开始。

众所周知，Javascript是全球最流行的语言之一，它涉足Web开发，移动端开发（[PhoneGap](http://phonegap.com/)、[Appcelerator](http://www.appcelerator.com/)），服务端开发（[Nodejs](http://nodejs.org/)、[Wakanda](http://wakanda.org/)），还有多种第三方实现（CoffeeScript这种）。此外Javascript还是许多开发者进入编程世界所接触的第一门语言。它既可以在浏览器中简单的弹出一个alert窗口，也能达到控制机器人这种复杂的程度（比如[nodebot](http://nodebots.io/)、[nodruino](http://semu.github.io/noduino/)）。现在那些能够熟练编写结构清晰、性能卓越的Javascript开发者们，已经成为招聘市场上炙手可热的应聘者了。

在这篇文章中，我将向你展示一系列Javascript相关的小技巧和一些最佳实践。除了少数几个示例，大部分示例都可以在浏览器环境或者服务端环境适用。

注意，文章中所有的代码片段都已经在Google Chrome V30(V8 3.20.17.15)测试通过。

# 使用`var`关键字进行变量赋值

在Javascript中，如果一个变量没有经过声明就直接进行赋值操作，那么这个变量就会自动转变成全局变量。我们要尽量避免这种情况（全局变量）。

# 使用`===`来代替`==`进行判等

`==`和`!=`操作符会在某些情况下自动进行类型转化。但是`===`和`!==`不会做自动转化，它们在做比较时，会同时比较数据类型和值，这也使得`===`和`!==`要比`==`和`!=`的速度要快。

```javascript
[10] === 10    // is false
[10]  == 10    // is true
'10' == 10     // is true
'10' === 10    // is false
[] == 0        // is true
[] ===  0      // is false
'' == false    // is true but true == "a" is false
'' ===   false // is false 
```

# `undefined`，`null`，0，`false`，`NaN`，`''`（空字符串）都为逻辑假值

# 使用分号`;`来结束一行代码

使用分号来结束代码行是一个被Javascript社区推荐的最佳实践。如果你忘了也没有关系，因为现在的Javascript引擎将会自动给你加上分号。至于我们为什么应该使用分号，可以参阅这篇文章[http://davidwalsh.name/javascript-semicolons](http://davidwalsh.name/javascript-semicolons)。

# 使用对象构造器

```javascript
function Person(firstName, lastName){
    this.firstName =  firstName;
    this.lastName = lastName;
}
var Saad = new Person("Saad", "Mousliki");
```

# 使用`typeof`、`instanceof`、`constructor`时要谨慎小心

- `typeof`：JavaScript的一元操作符，用于以字符串的形式返回变量的原始类型，注意，`typeof null`也会返回`object`，大多数的对象类型（数组`Array`、时间`Date`等）也会返回`object`
- `constructor`：对象（函数）的内部原型属性，它是可写的（可以被重写）
- `instanceof`：JavaScript操作符，会在原型链中的构造器中搜索，找到则返回`true`，否则返回`false`（常用于判断某一个对象是否是某个构造器或者其父类构造器的实例）

```javascript
var arr = ["a", "b", "c"];
typeof arr;   // return "object" 
arr instanceof Array // true
arr.constructor();  // []
```

# 学会使用自调用函数

函数在创建之后直接自动执行，通常称之为自调用匿名函数（Self-Invoked Anonymous Function）或直接调用函数表达式（Immediately Invoked Function Expression ）。比如，

```javascript
(function(){
    // some private code that will be executed automatically
})();

(function(a,b){
    var result = a+b;
    return result;
})(10,20);
```

# 随机从数组中取出一个元素

```javascript
var items = [12, 548 , 'a' , 2 , 5478 , 'foo' , 8852, , 'Doe' , 2145 , 119];
var randomItem = items[Math.floor(Math.random() * items.length)];
```

# 从一个指定的范围中取出一个随机数

这个功能在生成测试用的假数据时特别有用。比如取一个指定范围内的工资数。

```javascript
var x = Math.floor(Math.random() * (max - min + 1)) + min;
```

# 生成一个从0开始到指定数字的序列

```javascript
var numbersArray = [] , max = 100;
for( var i=1; numbersArray.push(i++) < max;);  // numbers = [1,2,3 ... 100] 
```

# 生成一个随机的字母数字序列

```javascript
function generateRandomAlphaNum(len) {
    var rdmString = "";
    for( ;rdmString.length < len; rdmString  += Math.random().toString(36).substr(2));
        return  rdmString.substr(0, len);
}
```

译者注

> `toString()`方法可以接受一个参数表示数字进制。而36进制刚好可以使用a-z和0-9这些字符。所以此方法可以用于生成简单的随机串。

# 打乱一个数字数组的顺序

```javascript
var numbers = [5, 458 , 120 , -215 , 228 , 400 , 122205, -85411];
numbers = numbers.sort(function(){ return Math.random() - 0.5});
/* the array numbers will be equal for example to [120, 5, 228, -215, 400, 458, -85411, 122205]  */
```

这里采用了原生的排序函数`sort()`，此外我们还可以使用专门的工具库来得到这一目的。

# 字符串去空格

像Java、C#、PHP这些语言都内置了`trim()`功能函数用于字符串去空格。但是Javascript没有这个内置方法。可以通过下面的方法来得到此目的，

```javascript
String.prototype.trim = function() {
    return this.replace(/^\s+|\s+$/g, "");
};
```

不过，在新的Javascript引擎中，已经内置支持了这个功能。

# 将一个数组追加到另一个数组中

```javascript
var array1 = [12 , "foo" , {name "Joe"} , -2458];
var array2 = ["Doe" , 555 , 100];
Array.prototype.push.apply(array1, array2);
/* array1 值为  [12 , "foo" , {name "Joe"} , -2458 , "Doe" , 555 , 100] */
```

# 将`argruments`转换成数组

```javascript
var argArray = Array.prototype.slice.call(arguments);
```

# 检验一个参数是否为数字

```javascript
function isNumber(n){
    return !isNaN(parseFloat(n)) && isFinite(n);
}
```

# 检验一个参数是否为数组

```javascript
function isArray(obj){
    return Object.prototype.toString.call(obj) === '[object Array]' ;
}
```

如果`toString()`被重写过的话，上面的方法就不行了。此时我们可以使用下面的方法，

```javascript
Array.isArray(obj); // its a new Array method
```

如果在浏览器中没有使用iframe，还可以用`instanceof`，但如果上下文太复杂，也有可能出错。比如，

```javascript
var myFrame = document.createElement('iframe');
document.body.appendChild(myFrame);

var myArray = window.frames[window.frames.length-1].Array;
var arr = new myArray(a,b,10); // [a,b,10]

// myArray 的构造器已经丢失，instanceof 的结果将不正常
// 不同iframe中的构造器是不能共享的
arr instanceof Array; // false
```

# 取出一个数组中的最大值和最小值

```javascript
var numbers = [5, 458 , 120 , -215 , 228 , 400 , 122205, -85411]; 
var maxInNumbers = Math.max.apply(Math, numbers); 
var minInNumbers = Math.min.apply(Math, numbers);
```

# 清空一个数组

```javascript
var myArray = [12 , 222 , 1000 ];  
myArray.length = 0; // myArray will be equal to [].
```

# 不用使用`delete`关键字来移除一个数组元素

应该使用`splice`方法而不是`delete`来移除一个数组元素。对一个数组元素使用`delete`会让这个数组元素的值变为`undefined`，并没有将这个数组元素给删除掉。

错误的用法，

```javascript
var items = [12, 548 ,'a' , 2 , 5478 , 'foo' , 8852, , 'Doe' ,2154 , 119 ]; 
items.length; // return 11 
delete items[3]; // return true 
items.length; // return 11 
/* items will be equal to [12, 548, "a", undefined × 1, 5478, "foo", 8852, undefined × 1, "Doe", 2154,       119]   */
```

正确的用法，

```javascript
var items = [12, 548 ,'a' , 2 , 5478 , 'foo' , 8852, , 'Doe' ,2154 , 119 ]; 
items.length; // return 11 
items.splice(3,1) ; 
items.length; // return 10 
/* items 结果为 [12, 548, "a", 5478, "foo", 8852, undefined × 1, "Doe", 2154, 119] */
```

注意，要想移除一个对象的属性，应该采用`delete`方法。


# 可以通过操作数组长度`length`来截断一个数组

与前面那个使用`length`清空数组的示例类似，我们可以使用`length`来截断一个数组。

```javascript
var myArray = [12 , 222 , 1000 , 124 , 98 , 10 ];  
myArray.length = 4; // myArray will be equal to [12 , 222 , 1000 , 124].
```

除此之外，如果我们使用一个更大的值去重写`length`，那么数组的长度将会改变，同时会用`undefined`填充新增的数组元素。

```javascript
myArray.length = 10; // the new array length is 10 
myArray[myArray.length - 1] ; // undefined
```

# 在条件中使用`&&`及`||`进行短语判断

```javascript
var foo = 10;  
foo == 10 && doSomething(); // is the same thing as if (foo == 10) doSomething(); 
foo == 5 || doSomething(); // is the same thing as if (foo != 5) doSomething();
```

`||`还用于给函数参数设置默认值，比如

```javascript
function doSomething(arg1){ 
    arg1 = arg1 || 10; // arg1 will have 10 as a default value if it’s not already set
}
```

# 使用`map()`对数组进行遍历操作

```javascript
var squares = [1,2,3,4].map(function (val) {  
    return val * val;  
}); 
// squares will be equal to [1, 4, 9, 16] 
```

# 保留指定位数的小数点

```javascript
var num = 2.443242342;
num = num.toFixed(4);  // num will be equal to 2.4432
```

注意，`toFixed()`方法返回的是字符串而不是一个数字。

# 浮点数计算问题

```javascript
0.1 + 0.2 === 0.3 // is false 
9007199254740992 + 1 // is equal to 9007199254740992  
9007199254740992 + 2 // is equal to 9007199254740994
```

为什么呢？因为0.1+0.2等于0.30000000000000004。JavaScript的数字都遵循IEEE 754标准构建，在内部都是64位浮点小数表示，具体可以参阅[这篇文章](http://www.2ality.com/2012/04/number-encoding.html)。

另外，你也可以使用`toFixed()`或者`toPrecision()`来解决这个问题。

译者注

> 关于这个问题，博主也有一篇相关的文章，[Javascript中浮点数的计算精度问题](http://gejiawen.github.io/2015/08/11/javascript-float-count-precision/)。


# 使用`for...in`来遍历对象的属性

下面的代码片段使用`for...in`来遍历对象属性，可以防止遍历到对象原型链上的属性。

```javascript
for (var name in object) {  
    if (object.hasOwnProperty(name)) { 
        // do something with name                    
    }  
}
```

# 逗号运算符

```javascript
var a = 0; 
var b = ( a++, 99 ); 
console.log(a);  // a will be equal to 1 
console.log(b);  // b is equal to 99
```

# 缓存临时变量用于避免再次计算或者查询

在使用jquery时，我们可以临时缓存整个jq对象，比如

```javascript
var navright = document.querySelector('#right'); 
var navleft = document.querySelector('#left'); 
var navup = document.querySelector('#up'); 
var navdown = document.querySelector('#down');
```

# 提前检查传入`isFinite()`的参数

```javascript
isFinite(0/0) ; // false 
isFinite("foo"); // false 
isFinite("10"); // true 
isFinite(10);   // true 
isFinite(undefined);  // false 
isFinite();   // false 
isFinite(null);  // true  注意这里！！！
```

# 避免对数组进行负值索引

```javascript
var numbersArray = [1,2,3,4,5]; 
var from = numbersArray.indexOf("foo") ;  // from is equal to -1 
numbersArray.splice(from,2);    // will return [5]
```

注意传给`splice`的索引参数不要是负数，当是负数时，会从数组结尾处删除元素。

# 使用`JSON`来进行序列化和反序列化

```javascript
var person = {name :'Saad', age : 26, department : {ID : 15, name : "R&D"} }; 
var stringFromPerson = JSON.stringify(person); 
/* stringFromPerson is equal to "{"name":"Saad","age":26,"department":{"ID":15,"name":"R&D"}}"   */ 
var personFromString = JSON.parse(stringFromPerson);  
/* personFromString is equal to person object  */
```

# 避免使用`eval()`和函数构造器

`eval()`和函数构造器（`Function` consturctor）的开销都比较大，每次调用JavaScript引擎都要将源代码转换为可执行的代码。

```javascript
var func1 = new Function(functionCode);
var func2 = eval(functionCode);
```

# 避免使用`with()`

使用`with()`语法会将变量注入到全局变量中。因此，如果有重名的变量，就会发生覆盖或者重写的问题。

# 避免使用`for...in`遍历数组

错误的用法，

```javascript
var sum = 0;  
for (var i in arrayNumbers) {  
    sum += arrayNumbers[i];  
}
```

更好的做法，

```javascript
var sum = 0;  
for (var i = 0, len = arrayNumbers.length; i < len; i++) {  
    sum += arrayNumbers[i];  
}
```

除此之外，`i`和`len`是在`for`循环的第一个声明中，二者只会初始化一次，这要比下面这种写法快：

```javascript
for (var i = 0; i < arrayNumbers.length; i++)
```

为什么呢？因为数组`arrayNumbers`的长度在每次遍历的时候都会计算一次，这就造成了不必要的消耗。

注意，这个问题其实在最新的Javascript引擎中已经被修复了。

# 给`setTimeout()`及`setInterval()`传递函数而不是字符串更好

如果你给`setTimeout()`或者`setInterval()`传递字符串的话，那么它内部的执行机制其实是和`eval()`是一样的，这样会比较慢。

错误的用法，

```javascript
setInterval('doSomethingPeriodically()', 1000);  
setTimeout('doSomethingAfterFiveSeconds()', 5000);
```

正确的用法，

```javascript
setInterval(doSomethingPeriodically, 1000);  
setTimeout(doSomethingAfterFiveSeconds, 5000);
```

# 使用`switch/case`来代替一坨`if/else`

当判断有超过两个分支的时候使用`switch/case`要更快一些，而且也更优雅，更利于代码的组织，当然，如果有超过10个分支，就不要使用`switch/case`了。

# 在`switch/case`中使用数字范围进行分界

其实`switch/case`中的`case`条件，还可以这样写：

```javascript
function getCategory(age) {  
    var category = "";  
    switch (true) {  
        case isNaN(age):  
            category = "not an age";  
            break;  
        case (age >= 50):  
            category = "Old";  
            break;  
        case (age <= 20):  
            category = "Baby";  
            break;  
        default:  
            category = "Young";  
            break;  
    };  
    return category;  
}  
getCategory(5);  // will return "Baby"
```

# 为创建的对象指定原型

下面的示例演示了可以给定对象作为参数，来创建以此为原型的新对象：

```javascript
function clone(object) {  
    function OneShotConstructor(){}; 
    OneShotConstructor.prototype = object;  
    return new OneShotConstructor(); 
} 
clone(Array).prototype ;  // []
```

# html转义函数

```javascript
function escapeHTML(text) {  
    var replacements= {'<': '&lt;', '>': '&gt;', '&': '&amp;', '"': '&quot;'};                      
    return text.replace(/[<>&"]/g, function(character) {  
        return replacements[character];  
    }); 
}
```

# 不要在循环中使用`try...catch...finally`

`try...catch...finally`在捕获一个异常时，会创建一个运行时环境的子作用域。而异常变量的生命周期仅限在这个运行时的子作用域。

译者注

> 这里我们可以使用闭包来保存这个运行时的异常变量。

错误的用法

```javascript
var object = ['foo', 'bar'], i;  
for (i = 0, len = object.length; i <len; i++) {  
    try {  
        // do something that throws an exception 
    } catch (e) {   
        // handle exception  
    } 
}
```

正确的用法

```javascript
var object = ['foo', 'bar'], i;  
try { 
    for (i = 0, len = object.length; i <len; i++) {  
        // do something that throws an exception 
    } 
} catch (e) {   
    // handle exception  
}
```

# 使用`XMLHttpRequests`时注意设置超时参数

如果一个ajax请求长时间没有响应，我们应该中止请求。否则浏览器将会一直等待。我们可以使用`setTimeout()`来做一个定时器，

```javascript
var xhr = new XMLHttpRequest (); 
xhr.onreadystatechange = function () {  
    if (this.readyState == 4) {  
        clearTimeout(timeout);  
        // do something with response data 
    }  
}  
var timeout = setTimeout( function () {  
    xhr.abort(); // call error callback  
}, 60*1000 /* timeout after a minute */ ); 
xhr.open('GET', url, true);  
xhr.send();
```

除此之外，我们应该避免同时发送多个**同步的**ajax请求。

# 处理websocket超时

一般地，WebSocket连接创建后，如果30秒内没有任何活动服务器端会对连接进行超时处理，防火墙也可以对单位周期内没有活动的连接进行超时处理。

为了防止超时中断，你需要每隔一段时间发送一个心跳数据（空字符串）以保持websocket连接。下面的两个方法一个用于周期的发送心跳数据保持连接，一个是取消心跳数据包。

```javascript
var timerID = 0;

function keepAlive() {
    var timeout = 15000;
    if (webSocket.readyState == webSocket.OPEN) {
        webSocket.send('');
    }
    timerId = setTimeout(keepAlive, timeout);
}

function cancelKeepAlive() {
    if (timerId) {
        cancelTimeout(timerId);
    }
}
```

`keepAlive()`方法应该被添加在`webSOcket`连接的`onOpen()`方法的最后，而`cancelKeepAlive()`添加在`onClose()`方法的最后。

# 记住：原生操作肯定比函数调用要效率高

比如，

错误的用法，

```javascript
var min = Math.min(a,b); 
A.push(v);
```

正确的用法，

```javascript
var min = a < b ? a : b; 
A[A.length] = v;
```

# 编码时注意保持代码的优雅格式，上生产环境前做一些压缩工作。

# Javascript是一门很吊的语言，这里还有很多资源，

- Code Academy JavaScript tracks: [http://www.codecademy.com/tracks/javascript](http://www.codecademy.com/tracks/javascript)
- Eloquent JavaScript by Marjin Haverbeke: [http://eloquentjavascript.net/](http://eloquentjavascript.net/)
- Advanced JavaScript by John Resig: [http://ejohn.org/apps/learn/](http://ejohn.org/apps/learn/)

