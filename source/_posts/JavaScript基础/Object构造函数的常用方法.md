---
title: Object构造函数的常用方法
date: 2019-1-16 11:39:01
tag: JavaScript核心
category: JavaScript
---
#### Object.creat() 返回一个包含指定原型对象的对象

>  Object.create(proto, [propertiesObject]);

- 首要功能：创建一个新对象
- 可选功能：为创建的新对象指定一个原型对象，内部原理其实是改变新对象的__proto__指向。
- 可选功能：为创建的新对象添加一些新的属性

传统的通过原型链继承，或者组合继承方法，都是把一个父类的实例赋给子类的原型对象,由于父类的实例的__proto__指向父类的原型对象，这样的话子类的原型上就有了父类实例的所有属性和父类的原型对象。

#### Object.assign() 合并对象

> Object.assign(target, ...sources)   // ES6

- MDN解释：用于将所有**可枚举属性**的值从一个或多个源对象复制到目标对象。它将返回目标对象。
    > 注：只要不设置 
- 首要功能：Object.assign是ES6新添加的接口，主要的用途是用来合并多个JavaScript的对象。
- 


- 关于Object.assign的深拷贝问题：
