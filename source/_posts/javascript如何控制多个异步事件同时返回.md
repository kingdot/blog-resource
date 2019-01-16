---
title: javascript如何控制多个异步事件同时返回
date: 2019-1-16 11:39:01
tag: JavaScript核心
category: JavaScript
---
#### 一、Promise.all()方法
#### 二、手动控制
```javascript
let countFunc = (function timerCount(count){
    let asycArr = [];
    return function(data){
        asycArr.push(data);
        if(asycArr.length === count){
            console.log(asycArr);
            return asycArr;
        }
    }
})(3);

let promise1 = new Promise((resolve,reject)=>{
    resolve(1);
}).then(data=>{
    countFunc(data);
});
let promise2 = new Promise((resolve,reject)=>{
    resolve(2);
}).then(data=>{
    countFunc(data);
});
let promise3 = new Promise((resolve,reject)=>{
    resolve(3);
}).then(data=>{
    countFunc(data);
});

```