---
title: JavaScript原型链
date: 2019-1-16 11:39:01
tag: JavaScript核心
category: JavaScript
---
- ECMAScript 中描述了原型链的概念，并将原型链作为实现继承的主要方法。其基本思想是利用原型让一个引用类型继承另一个引用类型的属性和方法。

- 简单回顾一下构造函数、原型和实例的关系：每个构造函数都有一个原型对象，原型对象都包含一个指向构造函数的指针，而实例都包含一个指向原型对象的内部指针。

- 那么，假如我们让原型对象等于另一个类型的实例，结果会怎么样呢？显然，此时的原型对象（另一个类的实例）将包含一个指向另一个原型的指针，相应地，另一个原型中也包含着一个指向另一个构造函数的指针。假如另一个原型又是另一个类型的实例，那么上述关系依然成立，如此层层递进，就构成了实例与原型的链条。这就是所谓原型链的基本概念。



#### 理清Function、Object的关系

- 首先明确：Function、Object都是构造函数，是函数就有prototype属性。
- 所有函数都是Function构造函数的实例，因此所有函数都是对象，即都有\_\_proto__属性。
- 所有对象都是Object的实例，且都有\_\_proto__属性。

- 由于Object是一个函数，即为Function的实例，因此有：
> Object.\_\_proto__ === Function.prototype

- 由于Function是一个函数，所有函数都是Function的实例，因此Function为一个对象,就有：
> Function.\_\_proto__ === Function.prototype
```
补充：
    一般情况下：obj.__proto__ === Obj.prototype
    由以上Function.__proto__ === Function.prototype知：
    Function的构造函数是Function，即Function自己构造了自己。
```
- 由以上得出：
> Object.\_\_proto__ === Functin.prototype === Function.\_\_proto__