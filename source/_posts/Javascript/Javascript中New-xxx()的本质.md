postid: "deep-into-new-object"
title: Javascript中New xxx()的本质
date: 2014-09-19 17:07:47
categories: [Javascript]
tags: [javascript]
---

在Javascript中，

```javascript
var a = new A();
```

它做了如下几件事，

- 创建一个空的对象`object`
- 把`object`绑定到函数`A`的上下文中（即A中的`this`现在指向`object`）
- 执行函数`A`
- 返回`object`

所以，`var a1 = new A()`与`var a2 = A()`这两句有着本质的区别！


