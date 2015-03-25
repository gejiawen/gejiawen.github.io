title: Javascript设计模式简述
date: 2014-09-23 11:42:43
categories: [大前端]
tags: [javascript, 设计模式, 转载]
---

本文系转载，来自[博客园-聂微东](http://www.cnblogs.com/Darren_code/)。

# 前言

- 单体模式，工厂模式，桥梁模式个人认为这个一个优秀前端必须掌握的模式，对抽象编程和接口编程都非常有好处。
- 装饰者模式和组合模式有很多相似的地方，它们都与所包装的对象实现同样的接口并且会把任何方法的调用传递给这些对象。装饰者模式和组合模式是本人描述的较吃力的两个模式，我个人其实也没用过，所以查了很多相关资料和文档，请大家海涵。
- 门面模式是个非常有意思的模式，几乎所有的JavaScript库都会用到这个模式，假如你有逆向思维或者逆向编程的经验，你会更容易理解这个模式（听起来有挑战，其实一接触你就知道这是个很简单的模式）；还有配置器模式得和门面模式一块拿来说，这个模式对现有接口进行包装，合理运用可以很多程度上提高开发效率。这两个模式有相似的地方，所以一块理解的话相信都会很快上手的。
- 享元模式是一种以优化为目的的模式。
- 代理模式主要用于控制对象的访问，包括推迟对其创建需要耗用大量计算资源的类得实例化。
- 观察者模式用于对对象的状态进行观察，并且当它发生变化时能得到通知的方法。用于让对象对事件进行监听以便对其作出响应。观察者模式也被称为“订阅者模式”。
- 命令模式是对方法调用进行封装的方式，用命名模式可以对方法调用进行参数化和传递，然后在需要的时候再加以执行。
- 职责链模式用来消除请求的发送者和接收者之间的耦合。


# JavaScript设计模式都有哪些？

## 单体（Singleton）模式

**绝对是JavaScript中最基本最有用的模式**

单体在JavaScript的有多种用途，它用来划分命名空间。可以减少网页中全局变量的数量(在网页中使用全局变量有风险)；可以在多人开发时避免代码的冲突(使用合理的命名空间)等等。

在中小型项目或者功能中，单体可以用作命名空间把自己的代码组织在一个全局变量名下；在稍大或者复杂的功能中，单体可以用来把相关代码组织在一起以便日后好维护。

使用单体的方法就是用一个命名空间包含自己的所有代码的全局对象，

```javascript
var functionGroup = {
    name:'Darren',
    method1:function(){
        //code
    },
    init:function(){
        //code
    }
};
```

或者

```javascript
var functionGroup = myGroup()
{
    this.name = 'Darren';
    this.getName = function () {
        return this.name
    };
    this.method1 = function () {
    };
    // ...
}
```

## 工厂（Factory）模式

**提供一个创建一系列相关或相互依赖对象的接口，而无需指定他们具体的类**

工厂就是把成员对象的创建工作转交给一个外部对象，好处在于消除对象之间的耦合(何为耦合？就是相互影响)。通过使用工厂方法而不是new关键字及具体类，可以把所有实例化的代码都集中在一个位置，有助于创建模块化的代码，这才是工厂模式的目的和优势。

举个例子，
你有一个大的功能要做，其中有一部分是要考虑扩展性的，那么这部分代码就可以考虑抽象出来，当做一个全新的对象做处理。好处就是将来扩展的时候容易维护 - 只需要操作这个对象内部方法和属性，达到了动态实现的目的。

非常有名的一个示例 - XHR工厂，

```javascript
var XMLHttpFactory = function () {
};
　　　　　　//这是一个简单工厂模式

XMLHttpFactory.createXMLHttp = function () {
    var XMLHttp = null;
    if (window.XMLHttpRequest) {
        XMLHttp = new XMLHttpRequest();
    }
    elseif(window.ActiveXObject)
    {
        XMLHttp = new ActiveXObject("Microsoft.XMLHTTP");
    }

    return XMLHttp;
};

//XMLHttpFactory.createXMLHttp()这个方法根据当前环境的具体情况返回一个XHR对象。
var AjaxHander = function () {
    var XMLHttp = XMLHttpFactory.createXMLHttp();
    // ...
};
```

工厂模式又区分简单工厂模式和抽象工厂模式，上面介绍的是简单工厂模式，这种模式用的更多也更简单易用。

抽象工厂模式的使用方法就是 - 先设计一个抽象类，这个类不能被实例化，只能用来派生子类，最后通过对子类的扩展实现工厂方法。 示例，

```javascript
var XMLHttpFactory = function () {};
//这是一个抽象工厂模式

XMLHttpFactory.prototype = {
    //如果真的要调用这个方法会抛出一个错误，它不能被实例化，只能用来派生子类
    createFactory: function () {
        throw new Error('This is an abstract class');
    }
};

//派生子类，
var XHRHandler = function () {
    XMLHttpFactory.call(this);
};

XHRHandler.prototype = new XMLHttpFactory();
XHRHandler.prototype.constructor = XHRHandler;

//重新定义createFactory 方法
XHRHandler.prototype.createFactory = function () {
    var XMLHttp = null;
    if (window.XMLHttpRequest) {
        XMLHttp = new XMLHttpRequest();
    } else if (window.ActiveXObject) {
        XMLHttp = new ActiveXObject("Microsoft.XMLHTTP");
    }

    return XMLHttp;
};
```

## 桥接（bridge）模式

**在实现API的时候，桥梁模式灰常有用。在所有模式中，这种模式最容易立即付诸实施**

桥梁模式可以用来弱化它与使用它的类和对象之间的耦合，就是将抽象与其实现隔离开来，以便二者独立变化；
这种模式对于JavaScript中常见的时间驱动的编程有很大益处，桥梁模式最常见和实际的应用场合之一是时间监听器回调函数。

先分析一个不好的示例，

```javascript
element.onclick = function () {
    new setLogFunc();
};
```

为什么说这个示例不好，因为从这段代码中无法看出那个`LogFunc`方法要显示在什么地方，它有什么可配置的选项以及应该怎么去修改它。

换一种说法就是，桥梁模式的要诀就是让接口“可桥梁”，实际上也就是可配置。把页面中一个个功能都想象成模块，接口可以使得模块之间的耦合降低。

掌握桥梁模式的正确使用收益的不只是你，还有那些负责维护你代码的人。把抽象于其实现隔离开，可独立地管理软件的各个部分，bug也因此更容易查找。

桥梁模式目的就是让API更加健壮，提高组件的模块化程度，促成更简洁的实现，并提高抽象的灵活性。

一个好的示例，

```javascript
element.onclick = function () {
    //API可控制性提高了，使得这个API更加健壮
    new someFunction(element, param, callback);
};
```

*Tips：桥梁模式还可以用于连接公开的API代码和私有的实现代码，还可以把多个类连接在一起。*

```javascript
// 错误的方式
// 这个API根据事件监听器回调函数的工作机制，事件对象被作为参数传递给这个函数。本例中并没有使用这个参数，而只是从this对象获取ID。

addEvent(element, 'click', getBeerById);
function getBeerById(e) {
    var id = this.id;
    asyncRequest('GET', 'beer.url?id=' + id, function (resp) {
        //Callback response
        console.log('Requested Beer: ' + resp.responseText);
    });
}


// 好的方式
// 从逻辑上分析，把id传给getBeerById函数式合情理的，且回应结果总是通过一个回调函数返回。
// 这么理解，我们现在做的是针对接口而不是实现进行编程，用桥梁模式把抽象隔离开来。

function getBeerById(id, callback) {
    asyncRequest('GET', 'beer.url?id=' + id, function (resp) {
        callback(resp.responseText);
    });
}

addEvent(element, 'click', getBeerByIdBridge);

function getBeerByIdBridge(e) {
    getBeerById(this.id, function (beer) {
        console.log('Requested Beer: ' + beer);
    });
}
```

## 装饰者（Decorator）模式

**这个模式就是为对象增加功能(或方法)。**

动态地给一个对象添加一些额外的职责。就扩展功能而言，它比生成子类方式更为灵活。

装饰者模式和组合模式有很多共同点，它们都与所包装的对象实现统一的接口并且会把任何方法条用传递给这些对象。可是组合模式用于把众多子对象组织为一个整体，而装饰者模式用于在不修改现有对象或从派生子类的前提下为其添加方法。

装饰者的运作过程是透明的，这就是说你可以用它包装其他对象，然后继续按之前使用那个对象的方法来使用，从下面的例子中就可以看出。

还是从代码中理解吧，

```javascript
// 创建一个命名空间为myText.Decorations
var myText = {};
myText.Decorations = {};

myText.Core = function (myString) {
    this.show = function () {
        return myString;
    }
};

// 第一次装饰
myText.Decorations.addQuestuibMark = function (myString) {
    this.show = function () {
        return myString.show() + '?';
    };
};

// 第二次装饰
myText.Decorations.makeItalic = function (myString) {
    this.show = function () {
        return'<li>' + myString.show() + '</li>'
    };
};

// 得到myText.Core的实例
var theString = new myText.Core('this is a sample test String');
alert(theString.show()); //output 'this is a sample test String'

theString = new myText.Decorations.addQuestuibMark(theString);
alert(theString.show()); //output 'this is a sample test String?'

theString = new myText.Decorations.makeItalic(theString);
alert(theString.show()); //output '<li>this is a sample test String</li>'
```

从这个示例中可以看出，这一切都可以不用事先知道组件对象的接口，甚至可以动态的实现，在为现有对象增添特性这方面，装饰者模式有极大的灵活性。

如果需要为类增加特性或者方法，而从该类派生子类的解决办法并不实际的话，就应该使用装饰者模式。派生子类之所以会不实际最常见的原因是需要添加的特性或方法的数量要求使用大量子类。


## 组合（Composite）模式

**将对象组合成树形结构以表示“部分-整体”的层次结构。它使得客户对单个对象和复合对象的使用具有一致性。**

组合模式是一种专为创建Web上的动态用户界面而量身定制的模式。使用这种模式，可以用一条命令在多个对象上激发复杂的或递归的行为。组合模式擅长于对大批对象进行操作。

组合模式的好处，

- 程序员可以用同样的方法处理对象的集合与其中的特定子对象
- 它可以用来把一批子对象组织成树形结构，并且使整棵树都可被便利

组合模式适用范围，

- 存在一批组织成某处层次体系的对象（具体结构可能在开发期间无法知道）
- 希望对这批对象或其中的一部分对象实话一个操作

其实组合模式就是将一系列相似或相近的对象组合在一个大的对象，由这个大对象提供一些常用的接口来对这些小对象进行操作，代码可重用，对外操作简单。例如：对form内的元素，不考虑页面设计的情况下，一般就剩下input了，对于这些input都有name和value的属性，因此可以将这些input元素作为form对象的成员组合起来，form对象提供对外的接口，便可以实现一些简单的操作，比如设置某个input的value，添加/删除某个input等等。

这种模式描述起来比较吃力，我从《JS设计模式》上找个一个实例，大家还是看代码吧。

先创建组合对象类，

```javascript
// DynamicGallery Class
var DynamicGallery = function (id) { // 实现Composite，GalleryItem组合对象类
    this.children = [];
    this.element = document.createElement('div');
    this.element.id = id;
    this.element.className = 'dynamic-gallery';
};

DynamicGallery.prototype = {
    // 实现Composite组合对象接口
    add: function (child) {
        this.children.push(child);
        this.element.appendChild(child.getElement());
    },
    remove: function (child) {
        for (var node, i = 0; node = this.getChild(i); i++) {
            if (node == child) {
                this.children.splice(i, 1);
                break;
            }
        }
        this.element.removeChild(child.getElement());
    },
    getChild: function (i) {
        return this.children[i];
    },
    // 实现DynamicGallery组合对象接口
    hide: function () {
        for (var node, i = 0; node = this.getChild(i); i++) {
            node.hide();
        }
        this.element.style.display = 'none';
    },
    show: function () {
        this.element.style.display = 'block';
        for (var node, i = 0; node = getChild(i); i++) {
            node.show();
        }
    },
    // 帮助方法
    getElement: function () {
        return this.element;
    }
};
```

再创建叶对象类，

```javascript
var GalleryImage = function (src) { // 实现Composite和GalleryItem组合对象中所定义的方法
    this.element = document.createElement('img');
    this.element.className = 'gallery-image';
    this.element.src = src;
};

GalleryImage.prototype = {
    // 实现Composite接口
    // 这些是叶结点，所以我们不用实现这些方法，我们只需要定义即可
    add: function () {
    },
    remove: function () {
    },
    getChild: function () {
    },
    // 实现GalleryItem接口
    hide: function () {
        this.element.style.display = 'none';
    },
    show: function () {
        this.element.style.display = '';
    },
    // 帮助方法
    getElement: function () {
        return this.element;
    }
};
```

现在我们可以使用这两个类来管理图片，

```javascript
var topGallery = new DynamicGallery('top-gallery');

topGallery.add(new GalleryImage('/img/image-1.jpg'));
topGallery.add(new GalleryImage('/img/image-2.jpg'));
topGallery.add(new GalleryImage('/img/image-3.jpg'));

var vacationPhotos = new DyamicGallery('vacation-photos');

for (var i = 0; i < 30; i++) {
    vacationPhotos.add(new GalleryImage('/img/vac/image-' + i + '.jpg'));
}

topGallery.add(vacationPhotos);
topGallery.show();
vacationPhotos.hide();
```

## 门面（facade）模式

**门面模式是几乎所有JavaScript库的核心原则**

子系统中的一组接口提供一个一致的界面，门面模式定义了一个高层接口，这个接口使得这一子系统更加容易使用，简单的说这是一种组织性的模式，它可以用来修改类和对象的接口，使其更便于使用。

门面模式的两个作用，

- 简化类的接口
- 消除类与使用它的客户代码之间的耦合

门面模式的使用目的就是图方便。

想象一下计算机桌面上的那些快捷方式图标，它们就是在扮演一个把用户引导至某个地方的接口的角色，每次操作都是间接的执行一些幕后的命令。

你在看这篇的博客的时候我就假设你已经有JavaScript的使用经验了，那么你一定写过或者看过这样的代码，

```javascript
var addEvent = function (el, type, fn) {
    if (window.addEventListener) {
        el.addEventListener(type, fn);
    } else if (window.attachEvent) {
        el.attachEvent('on' + type, fn);
    } else {
        el['on' + type] = fn;
    }
};
```

这个就是一个JavaScript中常见的事件监听器函数，这个函数就是一个基本的门面，有了它，就有了为DOM节点添加事件监听器的简便方法。

现在要说门面模式的精华部分了，为什么说JavaScript库几乎都会用这种模式类。假如现在要设计一个库，那么最好把其中所有的工具元素放在一起，这样更好用，访问起来更简便。看代码，

```javascript
//_model.util是一个命名空间
_myModel.util.Event = {
    getEvent: function (e) {
        return e || window.event;
    },
    getTarget: function (e) {
        return e.target || e.srcElement;
    },
    preventDefault: function (e) {
        if (e.preventDefault) {
            e.preventDefault();
        } else {
            e.returnValue = false;
        }
    }
};

//事件工具大概就是这么一个套路，然后结合addEvent函数使用
addEvent(document.getElementsByTagName('body')[0], 'click', function (e) {
    alert(_myModel.util.Event.getTarget(e));
});
```

个人认为，在处理游览器差异问题时最好的解决办法就是把这些差异抽取的门面方法中，这样可以提供一个更一致的接口，addEvent函数就是一个例子。


## 适配置器（Adapter）模式

**将一个类的接口转换成客户希望的另外一个接口**

配器模式使得原本由于接口不兼容而不能一起工作的那些类可以一起工作，使用这种模式的对象又叫包装器，因为他们是在用一个新的接口包装另一个对象。

从表面上看，它和门面模式有点相似，差别在于它们如何改变接口，门面模式展现的是一个简化的接口，它并不提供额外的选择，而适配器模式则要把一个接口转换为另一个接口，它并不会滤除某些能力，也不会简化接口。

先来一个简单的示例看看，

```javascript
//假如有一个3个字符串参数的函数，但是现在拥有的却是一个包含三个字符串元素的对象，那么就可以用一个配置器来衔接二者
var clientObject = {
    str1: 'bat',
    str2: 'foo',
    str3: 'baz'
};

function interfaceMethod(str1, str2, str3) {
    alert(str1);
}

//配置器函数
function adapterMethod(o) {
    interfaceMethod(o.str1, o.str2, o.str3);
}

//adapterMethod函数的作为就在于对interfaceMethod函数进行包装，并把传递给它的参数转换为后者需要的形式。
adapterMethod(clientObject);
```

适配器模式的工作机制是：**用一个新的接口对现有类得接口进行包装。**

示例：适配两个库。下面的例子要实现的是从Prototype库的$函数到YUI的get方法的转换。

```javascript
// 先看它们在接口方面的差别
// Prototype $ function

function $() {
    var elements = new Array();
    for (var i = 0; i < arguments.length; i++) {
        var element = arguments[i];
        if (typeof element == 'string') {
            element = document.getElementById(element);
        }
//        if (typeof.length == 1) {
//            return element;
//        }
        elements.push(element);
    }

    return elements;
}

// YUI get method
YAHOO.util.Dom.get = function (el) {
    if (YAHOO.lang.isString(el)) {
        return document.getElementById(el);
    }

    if (YAHOO.lang.isArray(el)) {
        var c = [];
        for (var i = 0, len = el.length; i < len; ++i) {
            c[c.length] = YAHOO.util.Dom.get(el[i]);
        }
        return c;
    }

    if (el) {
        return el;
    }

    return null;
};

// 二者区别就在于get具有一个参数，且可以是HTML,字符串或者数组；而$木有正是的参数，允许使用者传入任意数目的参数，不管HTML还是字符串。
// 如果需要从使用Prototype的$函数改为使用YUI的get方法（或者相反，那么用适配器模式其实很简单）
function PrototypeToYUIAdapter() {
    return YAHOO.util.Dom.get(arguments);
}

function YUIToPrototypeAdapter(el) {
    return $.apply(window, el instanceof Array ? el : [el]);
}
```

## 享元（Flyweight）模式

**运用共享技术有效地支持大量细粒度的对象。**

享元模式可以避免大量非常相似类的开销。在程序设计中有时需要生成大量细粒度的类实例来表示数据。如果发现这些实例除了几个参数外基本伤都是相同的，有时就能够受大幅度第减少需要实例化的类的数量。如果能把这些参数移到类实例外面，在方法调用时将他们传递进来，就可以通过共享大幅度地减少单个实例的数目。

从实际出发说说自己的理解吧。

组成部分，

- “享元”：抽离出来的外部操作和数据；
- “工厂”：创造对象的工厂；
- “存储器”：存储实例对象的对象或数组，供“享元”来统一控制和管理。

应用场景，

- 页面存在大量资源密集型对象；
- 这些对象具备一定的共性，可以抽离出公用的操作和数据

关键，

- 合理划分内部和外部数据。既要保持每个对象的模块性、保证享元的独立、可维护，又要尽可能多的抽离外部数据。
- 管理所有实例。既然抽离出了外部数据和操作，那享元就必须可以访问和控制实例对象。在JavaScript这种动态语言中，这个需求是很容易实现的：我们可以把工厂生产出的对象简单的扔在一个数组中。为每个对象设计暴露给外部的方法，便于享元的控制。

优点，

- 将能耗大的操作抽离成一个，在资源密集型系统中，可大大减少资源和内存占用；
- 职责封装，这些操作独立修改和维护；

缺点，

- 增加了实现复杂度。 将原本由一个工厂方法实现的功能，修改为了**一个享元** + **一个工厂** + **一个存储器**。
- 对象数量少的情况，可能会增大系统开销。


示例，

```javascript
//汽车登记示例
var Car = function (make, model, year, owner, tag, renewDate) {
    this.make = make;
    this.model = model;
    this.year = year;
    this.owner = owner;
    this.tag = tag;
    this.renewDate = renewDate;
}

Car.prototype = {
    getMake: function () {
        return this.make;
    },
    getModel: function () {
        return this.model;
    },
    getYear: function () {
        return this.year;
    },
    transferOwner: function (owner, tag, renewDate) {
        this.owner = owner;
        this.tag = tag;
        this.renewDate = renewDate;
    },
    renewRegistration: function (renewDate) {
        this.renewDate = renewDate;
    }
}

//数据量小到没多大的影响，数据量大的时候对计算机内存会产生压力，下面介绍享元模式优化后
//包含核心数据的Car类

var Car = function (make, model, year) {
    this.make = make;
    this.model = model;
    this.year = year;
};

Car.prototype = {
    getMake: function () {
        return this.make;
    },
    getModel: function () {
        return this.model;
    },
    getYear: function () {
        return this.year;
    }
}

//中间对象，用来实例化Car类
var CarFactory = (function () {
    var createdCars = {};

    return {
        createCar: function (make, model, year) {
            var car = createdCars[make + "-" + model + "-" + year];
            return car ? car : createdCars[make + '-' + model + '-' + year] = (new Car(make, model, year));
        }
    };
})();

//数据工厂，用来处理Car的实例化和整合附加数据
var CarRecordManager = (function () {
    var carRecordDatabase = {};

    return {
        addCarRecord: function (make, model, year, owner, tag, renewDate) {
            var car = CarFactory.createCar(make, model, year);
            carRecordDatabase[tag] = {
                owner: owner,
                tag: tag,
                renewDate: renewDate,
                car: car
            }
        },
        transferOwnership: function (tag, newOwner, newTag, newRenewDate) {
            var record = carRecordDatabase[tag];
            record.owner = newOwner;
            record.tag = newTag;
            record.renewDate = newRenewDate;
        },
        renewRegistration: function (tag, newRenewDate) {
            carRecordDatabase[tag].renewDate = newRenewDate;
        },
        getCarInfo: function (tag) {
            return carRecordDatabase[tag];
        }
    };
})();
```

## 代理（Proxy）模式

**此模式最基本的形式是对访问进行控制**

代理对象和另一个对象（本体）实现的是同样的接口，可是实际上工作还是本体在做，它才是负责执行所分派的任务的那个对象或类，代理对象不会在另以对象的基础上修改任何方法，也不会简化那个对象的接口。

举一个具体的情况：如果那个对象在某个远端服务器上，直接操作这个对象因为网络速度原因可能比较慢，那我们可以先用Proxy来代替那个对象。

总之对于开销较大的对象，只有在使用它时才创建，这个原则可以为我们节省很多内存。《JS设计模式》上的图书馆示例，

```javascript
var Publication = new Interface('Publication', ['getIsbn', 'setIsbn', 'getTitle', 'setTitle', 'getAuthor', 'setAuthor', 'display']);

var Book = function (isbn, title, author) {
    //...
};

// implements Publication
implements(Book, Publication);

/* Library interface. */
var Library = new Interface('Library', ['findBooks', 'checkoutBook', 'returnBook']);

/* PublicLibrary class. */
var PublicLibrary = function (books) {
    //...
};

// implements Library
implements(PublicLibrary, Library);

PublicLibrary.prototype = {
    findBooks: function (searchString) {
        //...
    },
    checkoutBook: function (book) {
        //...
    },
    returnBook: function (book) {
        //...
    }
};

/* PublicLibraryProxy class, a useless proxy. */
var PublicLibraryProxy = function (catalog) {
    this.library = new PublicLibrary(catalog);
};

// implements Library
implements(PublicLibraryProxy, Library);

PublicLibraryProxy.prototype = {
    findBooks: function (searchString) {
        return this.library.findBooks(searchString);
    },
    checkoutBook: function (book) {
        return this.library.checkoutBook(book);
    },
    returnBook: function (book) {
        return this.library.returnBook(book);
    }
};
```

## 观察者（Observer）模式观察者（Observer）模式

**定义对象间的一种一对多的依赖关系，以便当一个对象的状态发生改变时，所有依赖于它的对象都得到通知并自动刷新。**

观察者模式中存在两个角色,观察者和被观察者。在DOM的编程环境中的高级事件模式中，*事件监听器说到底就是一种内置的观察者*。事件处理器(handler)和时间监听器(listener)并不是一回事，前者就是一种把事件传给与其关联的函数的手段，而在后者中，一个时间可以与几个监听器关联，每个监听器都能独立于其他监听器而改变。

```javascript
//使用时间监听器可以让多个函数相应一个事件
var fn1 = function () {
    //code
};
var fn2 = function () {
    //code
};

addEvent(element, 'click', fn1);
addEvent(element, 'click', fn2);

//而时间处理函数就办不到
element.onclick = fn1;
element.onclick = fn2;
```

观察者模式是开发基于行为的应用程序的有力手段，前端程序员可做的就是借助一个事件监听器替你处理各种行为，从而降低内存消耗和提高互动性能。


## 命令（Command）模式

**将一个请求封装为一个对象，从而使你可用不同的请求对客户进行参数化；对请求排队或记录请求日志，以及支持可取消的操作。**

命令对象是一个操作和用来调用这个操作的对象的结合体，所有的命名对象都有一个执行操作，其用途就是调用命令对象所绑定的操作。

示例，

```javascript
var Calculator = {
    add: function (x, y) {
        return x + y;
    },
    substract: function (x, y) {
        return x - y;
    },
    multiply: function (x, y) {
        return x * y;
    },
    divide: function (x, y) {
        return x / y;
    }
};

Calculator.calc = function (command) {
    return Calculator[command.type](command.op1, command.opd2)
};

Calculator.calc({type: 'add', op1: 1, op2: 1});
Calculator.calc({type: 'substract', op1: 5, op2: 2});
Calculator.calc({type: 'multiply', op1: 5, op2: 2});
Calculator.calc({type: 'divide', op1: 8, op2: 4});
```

命名模式的主要用途是把调用对象（用户界面，API和代理等）与实现操作的对象隔离开，也就是说使对象间的互动方式需要更高的模块化时都可以用到这种模式。


## 职责链（Chain Of Responsibility）模式

**为解除请求的发送者和接收者之间耦合，而使多个对象都有机会处理这个请求。将这些对象连成一条链，并沿着这条链传递该请求，直到有一个对象处理它。**

职责链由多个不同类型的对象组成：发送者是发出请求的对象，而接收者则是接收请求并且对其进行处理或传递的对象，请求本身有时也是一个对象，它封装着与操作有关的所有数据。

典型的流程大致是，

- 发送者知道链中第一个接收者，它向这个接收者发出请求。
- 每一个接收者都对请求进行分析，然后要么处理它，要么将其往下传。
- 每一个接收者知道的其他对象只有一个，即它在链中的下家。
- 如果没有任何接收者处理请求，那么请求将从链上离开，不同的实现对此也有不同的反应，一般会抛出一个错误。

职责链模式的适用范围，

- 有多个的对象可以处理一个请求，哪个对象处理该请求运行时刻自动确定
- 想在不明确指定接收者的情况下，向多个对象中的一个提交一个请求
- 可处理一个请求的对象集合需要被动态指定。

*确实对这种模式不了解，相关资料也较少，所以代码先不上了。看看大家对这个模式有木有什么好的理解或者能较好表达这种模式的代码，谢谢了。*


# 结束语

- 每种模式都有自己的优缺点，所以每种模式的正确使用还得看开发人员本身的功力；
- 就算不使用JavaScript设计模式一样可以写出**复杂的可使用**的代码，可是如果你想真正了解JavaScript面向对象能力，学习提高代码的模块化程度﹑可维护性﹑可靠性和效率，那么合理的运用JavaScript设计模式将会是一个优秀**前端开发工程师**必备的能力。


End. All rights reserved `@gejiawen`.
