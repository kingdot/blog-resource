---
title: JS对象的等号赋值问题
date: 2019-1-16 11:39:01
tag: JavaScript核心
category: JavaScript
---
##### 首先明确两点，不管是作为函数参数还是等号赋值：基本类型是值传递，引用类型是地址传递

##### 举个栗子：

```
 let obj1 = {
     key1:1
 }
 let obj2 = obj1;
 obj2 === obj1 // true
 
 obj1 = null;
 console.log(obj1); // null
 console.log(obj2); // {key1:1}
 
 obj1 = obj2;
 obj1.key1 = 2;
 console.log(obj2); // {key1:2}
```

##### 区别：操作引用类型时，给变量重新赋值和修改变量所指向地址的内容，性质和效果都不一样！！！

前者无副作用，后者有！！！