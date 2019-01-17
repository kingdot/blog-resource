---
title: Promise详解
date: 2019-1-17 11:45:56
tag: ES6，ES7新特性
category: ES6
---
## 作用
异步解决方案。

## 使用方法:
**核心思想：通过异步事件自己主动调用resolve，reject方法驱动then的执行**
```javascript
let promise1 = new Promise(function(resolve,reject){
    if(dosomething){
        resolve(some);
    }else{
        reject(other);
    }
});

promise1.then(res=>{},rej=>{});
```
## 实现
### 构造函数的实现
```javascript
function MyPromise(excutor){
    let self = this;
    self.status = 'pending';
    self.data = undefined;
    // 存放每一个then里面的回调函数
    self.resolveCallbacks = [];
    self.rejectCallbacks = [];
    function resolve(data){ // 调了resolve方法之后，status才会变成resolved
        if(self.status === 'pending'){
            self.status = 'resolved';
            self.data = data;
            // 执行then里面的onfulfilled回调
            self.resolveCallbacks.forEach(func=>func(self.data));
        }
    }
    function reject(reason){
        if(self.status === 'pending'){
            self.status = 'rejected';  // 调了reject方法之后，status才会变成rejected
            self.data = reason;
            // 执行then里面的onRejected回调
            self.resolveCallbacks.forEach(func=>func(self.data)); // 才会执行then里面的回调
        }
    }
    
    try{
        excutor(resolve,reject);
    }catch(e){
        reject(e);
    }
}
```

### then的实现
#### 1. then也是一个函数，它接受两个函数参数。因此promise的执行过程为：
```
1. 先执行new：在这个过程中，传递给构造函数的excutor会被立即执行，返回一个可以被thenable的promise对象
2. 如果这个promise对象跟着的有then方法，那么在then(res=>{},rej=>{})的时候就执行了then方法，并不是类似事件监听机制，而是纯粹的函数执行。then的执行过程见下面then的实现。
```

#### 2. 由于可以多次then，且promise的状态一旦改变就不能再次改变，因此每次then要返回一个新的promise。

```javascript
MyPromise.prototype.then = function(onFulfilledCall, onRejectedCall){
    let self = this;
    // 根据标准，如果then的参数不是function，则我们需要忽略它，实现透明传输，以如下方式补上带有return的回调（手工return）
    onFulfilledCall = typeof onFulfilledCall === 'function' ? onFulfilledCall : val=>val;
    onRejectedCall = typeof onRejectedCall === 'function' ? onRejectedCall : err=>throw err;
    // 如果当前的promise状态已经resolve，此时再then，则then里面的onFulfilled回调函数立即获取到值并执行
    if(self.status === 'resolved'){
        // then方法的返回值，返回一个新的promise
        return promise2 =  new MyPromise(function(resolve, reject){
            // 为了保证then的顺序执行，这里用setTimeout包裹回调的执行
            setTimeout(()=>{
                // 因为考虑到有可能throw，所以我们将其包在try/catch块里
                try{
                    let x = onFulfilledCall(self.data);
                    // 因为then的回调函数执行结果可能是一个普通值，也可能是一个promise，需要分开处理
                    // 当x为普通值时，直接将x作为promise2成功（then）的结果，此次then结束。
                    // 当x为一个promise时，将x的结果（成功/失败）作为promise2成功/失败的结果
                    // 目的是保证当前then完全结束后，才继续后面的then
                    // 
                    resolvePromise(promise2,x,resolve,reject);
                }catch(e){
                    reject(e);
                }
                // resolve(x);  因为x不确定类型，因此这里不能直接resolve
            },0)
            
        })
    }
    // 如果当前的promise状态已经reject，此时再then，则then里面的OnRejected回调函数立即获取到值并执行
    if(self.status === 'rejected'){
        return promise2 =  new MyPromise(function(resolve, reject){
            // 因为考虑到有可能throw，所以我们将其包在try/catch块里
            try{
                let x = onRejectedCall(self.data);
                resolvePromise(promise2,x,resolve,reject);
            }catch(e){
                reject(e);
            }
            // reject(x);
        })
    }
    // 如果当前的Promise还处于pending状态，此时来个then，我们并不能确定调用onResolved还是onRejected，
    // 只能等到Promise的状态确定后，才能确实如何处理。
    // 所以我们需要把我们的**两种情况**的处理逻辑做为callback放入promise1(此处即this/self)的回调数组里
    // 逻辑本身跟第一个if块内的几乎一致，此处不做过多解释
    if(self.status === 'pending'){
        return promise2 = new Promise(function(resolve, reject) {
            // 之所以不直接push(onFulfilledCall),同样是因为onFulfilledCall的执行结果可能是一个promise，这时需要单独处理，因此不能简单的push(onFulfilledCall)
            self.resolveCallbacks.push(function(){
                // 因为考虑到有可能throw，所以我们将其包在try/catch块里
                try{
                    let x = onFulfilledCall(self.data);
                    resolvePromise(promise2,x,resolve,reject);
                }catch(e){
                    reject(e);
                }
                //resolve(x);
            });
            
            self.rejectCallbacks.push(function(){
                // 因为考虑到有可能throw，所以我们将其包在try/catch块里
                try{
                    let x = onRejectedCall(self.data);
                    resolvePromise(promise2,x,resolve,reject);
                }catch(e){
                    reject(e);
                }
                //reject(x);
            });
        }
    }
}

// 用于处理x和promise2的关系: 
function resolvePromise(promise2,x,resolve){
    // 不能自己等待自己成功或者失败
    if(x === promise2){
        return reject(new TypeError('Chaining cycle'));
    }
    let called;
    // x为对象，则看它是不是具有then方法
    if(x !== null && (typeof x === 'object' || typeof x === 'function')){
        try{ // 为了防止有人通过defineProperty定义then，且在getter里面抛出异常(即取then时报异常)，因此需要try...catch
            let then = x.then;
            if(typeof then === 'function'){ // x是个可以then的promise，取它的结果
                then.call(x, rs=>{
                    if(called) return；
                    called = true;
                    resolvePromise(promise2,rs,resolve,reject); // 取x的成功结果rs，继续进行判断
                },err=>{
                    if(called) return;
                    called = true;
                    reject(err); // 取x的失败结果，直接拒绝，不再继续判断
                })
            }else{
                resolve(x); // x是个具有then属性而不是then方法的对象
            }
        }
    }else{
        resolve(x); // x为普通值，直接resolve
    }
}

// 为了下文方便，我们顺便实现一个catch方法
Promise.prototype.catch = function(onRejected) {
    // 默认不写成功的接收
    return this.then(null, onRejected)
}   

// reject方法
Promise.reject = function(reason){
    return new Promise((resolve,reject)=>{
        reject(reason);
    })
}
// resolve方法
Promise.resolve = function(value){
    return new Promise((resolve,reject)=>{
        resolve(value);
    })
}
// Promise.all方法
Promise.all = function(promises){
    return new Promise((resolve,reject)=>{
        let arr = [];
        promises.forEach((promise,index)=>{
            promise.then(data=>{
                arr.push(data);
                if(index === promise.length-1){ // 全部返回后才resolve
                    resolve(arr);
                }
            })
        })
    })
}
// Promise.race
Promise.race = function(promises){
    return new Promise((resolve,reject)=>{
        promises.forEach((promise,index)=>{
            promise.then(
            // 错误的方法，因为此时的resolve和reject方法是promise的（执行时确定），而不是return的那个promise的
            /*data=>{
                resolve(data); 
            },reason=>{
                reject(reason);
            }*/
            // 正确的方法,需要调用外层return的promise的resolve和reject方法（作为参数传递）（这俩一旦被调用就表明promise有结果了，并且promise的成功/失败值会自动传入给resolve，reject方法）
            resolve,reject
            )
        })
    })
}
```

