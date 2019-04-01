---
title: Symbol相关
date: 2019-2-16 11:39:01
tag: JavaScript进阶
category: JavaScript
---

参考：[Symbol相关](https://blog.csdn.net/luofeixiongsix/article/details/78973801)

#### ES6引入了Symbol类型，即js中的第7种数据类型

#### 引入目的：每个从Symbol()返回的symbol值都是唯一的。一个symbol值能作为对象属性的标识符；这是该数据类型仅有的目的。

#### symbol分为全局和布局，分别用Symbol.for()和Symbol()产生，不支持new, 一般都使用局部的

```javascript
var a=Symbol("a"), b=Symbol("a");
a === b;    //false

var a = Symbol.for("a"), b = Symbol.for("a");
a === b;    //true
```

#### Symbol类型的key一般情况下不会被遍历到，除非使用Object.getOwnPropertySymbols();因此可以做到隐藏私有属性

#### 如何通过symbol隐藏私有属性？

> 首先，私有属性的key都用symbol去设置

> 然后，不对外暴漏产生这些symbol的声明，可以通过闭包实现，因为在闭包外部访问不到内部的声明

```javascript
// 隐藏了私有属性name
var People = (function() {
  var name = Symbol("name");
  class People {
    constructor(n) { //构造函数
    this[name] = n;
    }
    sayName() {
        console.log(this[name]);
    }
  }
  return People;
})();
```