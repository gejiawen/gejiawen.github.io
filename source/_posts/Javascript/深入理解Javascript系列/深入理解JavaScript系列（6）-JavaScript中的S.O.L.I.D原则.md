postid: "deep-into-javascript-solid-rules"
title: 深入理解JavaScript系列（6）-JavaScript中的S.O.L.I.D原则
date: 2014-12-30 18:38:03
categories: [Javascript]
tags: [深入理解JavaScript系列]
---

本文是[深入理解JavaScript系列](http://gejiawen.github.io/2014/11/13/%E7%B3%BB%E5%88%97/%E6%B7%B1%E5%85%A5%E7%90%86%E8%A7%A3JavaScript%E7%B3%BB%E5%88%97/)的第**六**篇读文笔记，博客原文如下，

- [深入理解JavaScript系列（6）：S.O.L.I.D五大原则之单一职责SRP](http://www.cnblogs.com/TomXu/archive/2012/01/06/2305513.html)
- [深入理解JavaScript系列（7）：S.O.L.I.D五大原则之开闭原则OCP](http://www.cnblogs.com/TomXu/archive/2012/01/09/2306329.html)
- [深入理解JavaScript系列（8）：S.O.L.I.D五大原则之里氏替换原则LSP](http://www.cnblogs.com/TomXu/archive/2012/01/10/2310244.html)
- [深入理解JavaScript系列（21）：S.O.L.I.D五大原则之接口隔离原则ISP](http://www.cnblogs.com/TomXu/archive/2012/02/14/2330137.html)
- [深入理解JavaScript系列（22）：S.O.L.I.D五大原则之依赖倒置原则DIP](http://www.cnblogs.com/TomXu/archive/2012/02/15/2330143.html)


我们先来看下，什么叫[**S.O.L.I.D原则**](http://en.wikipedia.org/wiki/SOLID_(object-oriented_design)？

五大原则的含义如下，

1. The Single Responsibility Principle（单一职责SRP），一个类的功能或者说职责应该是单一的。换句话说，一个类只允许一个原因能够让它发生变化。
2. The Open/Closed Principle（开闭原则OCP），一个类或者软件实体应该对扩展开放，对修改关闭。即应该在不修改的情况下即可扩展。
3. The Liskov Substitution Principle（里氏替换原则LSP），派生的对象或类型必需能够替换其基类。
4. The Interface Segregation Principle（接口分离原则ISP），不应该强迫客户依赖没有使用的接口。
5. The Dependency Inversion Principle（依赖反转原则DIP），我们应该依赖抽象，而不是依赖具体实现；而且低层实现应该依赖高层概念。

然后大叔针对每一个原则一连写了五篇文章。这些文章貌似都是来自同一个外国博客的翻译，但是现在这个外国原文已经打不开了。

这五篇文章其实质量不是很高，而且在我看来说的内容有点不伦不类。

在之前的相当一段时间内，JavaScript被大肆用来模拟面向对象编程，Yahoo的YUI框架在这件事上玩到了极致。除了YUI这种非常出名的框架之外，还有许多不知名的框架和类库在做着~~使用JavaScript来模拟OOP编程~~这件事。一时间轮子满天飞。

其实准确的来说，JavaScript是一门**基于对象**，采用**原型链方式**继承的语言，现在的OOP语言基本上都是采用的**基于类继承**的方式，从这一点上来说，JavaScript的继承机制对一部分人的理解造成了干扰。

传统的OOP语言（比如Java），我觉得基于类的对象继承可能并不是万能的，所以设计模式在Java这类编程语言中是一个非常重要的角色，特别是当你开发比较大型复杂的应用时。但是这并不代表我们需要将这些概念和模式生硬的照搬到JavaScript中。其实在某些场景中，JavaScript的原型机制可以表现的更好。

如果从语言的层面来说，JavaScript是一门多范式的语言，你既可以模仿C语言的风格来做面向过程式的编程，你也可以模仿面向对象的编程，而且你还可以享受函数式的编程体验，具体怎么去使用要看个人的能力和具体的场景。往往我们使用JavaScript开发应用的时候，不自觉的会用上多种编程风格，所以它是很灵活的，关键在于使用的人。

所以，大叔的这五篇文章我并不打算逐一的去分析了，我觉得把传统OOP语言的东西生搬硬套到JavaScript中并没有太大的意义，关键是看具体的场景和使用的人吧。

不过JavaScript也是有设计模式这种东西的，后面我会带来相应的文章。

