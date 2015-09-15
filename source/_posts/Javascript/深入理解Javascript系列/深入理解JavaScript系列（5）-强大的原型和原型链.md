postid: "deep-into-javascript-prototype-and-prototype-chain"
title: 深入理解JavaScript系列（5）-强大的原型和原型链
date: 2014-12-29 20:53:27
categories: [Javascript]
tags: [深入理解JavaScript系列]
---

本文是[深入理解JavaScript系列](http://gejiawen.github.io/2014/11/13/%E7%B3%BB%E5%88%97/%E6%B7%B1%E5%85%A5%E7%90%86%E8%A7%A3JavaScript%E7%B3%BB%E5%88%97/)的第**五**篇读文笔记，博客原文在[这里](http://www.cnblogs.com/TomXu/archive/2012/01/05/2305453.html)。

本篇的内容其实就是介绍JavaScript的一大核心内容**原型以及原型链**。

我们知道JavaScript应该是唯一一款被广泛使用的基于原型继承的语言。基于原型的继承跟传统的基于类继承有一些区别，从严格意义上来说，实现原型继承要比实现类继承要困难一些。

由于种种原因，以我现在的眼光来看，大叔当年写的这篇关于原型及原型链的文章的质量并不是很高，而且还有不少并不是最佳实践的地方。

那么问题来了，既然原文的质量都不高了，那我的读文笔记咋办呢？

注意了，下面有一个传送门，是我之前转载翻译的一遍国外大牛写的博客，当然内容也是阐述JavaScript原型及原型链相关的。

[**传送门**](http://gejiawen.github.io/2014/10/16/Javascript/%E6%B5%85%E8%B0%88Javascript%E4%B8%AD%E7%9A%84%E5%8E%9F%E5%9E%8B%E7%BB%A7%E6%89%BF/)

我个人觉得这篇外国友人的文章质量还是相当高的，而且通过循循渐进的案例分析，非常清晰的阐述了原型以及原型继承，真是让人欲罢不能，每日必读三遍。

啊，本篇读文笔记好水！！

