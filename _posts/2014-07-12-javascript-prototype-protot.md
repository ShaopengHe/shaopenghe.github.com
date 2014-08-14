---
title: 简析Javascript的prototype和__proto__
layout: post
tags:
    - js
    - javascript
---

首先，需要知道以下事实：

- Javascript的继承是基于原型链模式的。
- Javascript的原型是一个对象。
- Javascript的继承本质是一（或多）个对象引用了另一个对象的属性和方法。

**直接结论：**

**\__proto__ 是对象的一个内部隐藏属性，它的值是该对象的原型。**

**prototype 是函数对象的一个属性，它的值是一个对象（默认为 {} ），当该函数用作构造函数时，新创建对象的原型（\__proto__）将指向该属性值。**


好，show you the code:

```
> var A = function(x){this.name = x;};   //定义函数 A
> var a = new A('ima');     // 通过函数A创建对象a
> a
{ name: 'ima' }

> A.prototype       //因为A.prototype默认是 {}
{}
> a.__proto__       //所以对象a的原型（__proto__）就是 {}
{}
> A.prototype === a.__proto__   //它们都是指向同一个 {}
true
> {} === {}             //js中不同的空对象引用不一样，请对比上者感受一下
false
```

再展示另一个例子：

```
> var D = function(y){this.desc = y;};   //定义函数D

> D.prototype = a;           //将D的prototype指向对象a
{ name: 'ima' }

> var d = new D('imd')      //通过函数D创建对象d
> d
{ desc: 'imd' }

> d.__proto__               //对象d的原型就是对象a
{ name: 'ima' }
> d.__proto__ === a
true

> d.name                    //d"继承"了对象a的属性，实质是引用
'ima'
> a.name = 'iama!'          //更改a.name, d.name也发生了改变
> d.name                    //具体可以查看Javascript原型链原理
'iama!'

/* 需要注意的是：非函数对象是没有prototype属性的：*/
> a.prototype
undefined
> d.prototype
undefined

/* 而函数是有__proto__的，因为函数也是对象：*/
> A.__proto__      
[Function: Empty]       //函数的默认原型是空函数，正如对象的默认原型是空对象
> A.__proto__.toString()   
'function Empty() {}'
```

再次总结：

1. \__protot__指向对象的原型。
2. prototype是函数的属性，作为使用new创建的函数实例的原型。


建议阅读阮一峰的[Javascript继承机制的设计思想](http://www.ruanyifeng.com/blog/2011/06/designing_ideas_of_inheritance_mechanism_in_javascript.html)，简单通俗地了解为什么Javascript采用prototype这种方式实现继承。

----------

题外话：（关于constructor）

```
/* 对象a的构造函数是函数A */
> a.constructor.toString()
'function (x){this.name = x;}'
> a.constructor === A
true

/* 因为A是一个函数对象，所以A的构造函数就是function的构造函数Function */
> A.constructor
[Function: Function]
> A.constructor.toString()
'function Function() { [native code] }'
> A.constructor === Function.constructor
true
/* 函数A的原型(__protot__)就是它的构造函数（Function）的prototype */
> A.constructor === Function
true
> Function.prototype
[Function: Empty]
> A.__proto__ === Function.prototype
true

/* 但是 ！！！ */
/* 继承了对象a的d，它的构造函数不是D，而是A */
> d.constructor.toString()
'function (x){this.name = x;}'

> d.constructor === a.constructor
true

> d.constructor === A
true
```

以上代码在Node.js v0.10.30环境下测试。

