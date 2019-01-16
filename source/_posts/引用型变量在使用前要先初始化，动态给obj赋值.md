---
title: 引用型变量在使用前要先初始化，动态给obj赋值
date: 2019-1-16 11:39:01
tag: JavaScript核心
category: JavaScript
---
#### 引用型变量在使用前要初始化，否则它的方法不能使用；
```
let obj;
obj.push(1); // error: push is not defined in obj;
//=================================================
obj = [];
obj.push(1); // ok
```

#### 动态给obj赋值,实现类似Map的set(key, valye)功能
```
let obj = {};
let 
obj[key] = value;
```