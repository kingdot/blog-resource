---
title: JavaScript返回一个函数（闭包）的使用场景
date: 2019-1-16 11:39:01
tag: JavaScript核心
category: JavaScript
---

#### 1. 在内存中维持变量：如果缓存数据、柯里化。

#### 2. 批量制造函数或者需要返回一个函数的时候

#### 3. 循环绑定事件: 

```javascript
let arr = [];
for (var i = 0; i < 5; i++) {
    arr.push(() => {
        console.log(i)
    })
}
arr.forEach(f => f()); // 全输出5

// ====================================
// 先执行new，再执行`.`运算符
new Array(5).forEach((item,index) => {
    arr.push(() => {
        console.log(index)
    })
})
arr.forEach(f => f()); // 0，1，2，3，4
```

因此不建议在 `for` 循环中建立函数，而是使用数组的 `forEach` 方法，但是不兼容 IE678 ，需要额外添加 forEach 方法；
