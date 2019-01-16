---
title: EventLoop总结
date: 2019-1-16 11:39:01
tag: JavaScript核心
category: JavaScript
---
## 事件循环背景：

首先，js是一门单线程非阻塞的语言。为了协调事件（event），用户交互（user interaction），脚本（script），渲染（rendering），网络（networking）等，用户代理（user agent）必须使用事件循环（event loops）。

主线程执行代码的过程中，一旦遇到异步操作，就将其挂起，交给宿主环境中的对应异步线程去执行这个事件。

## 一、浏览器上的EventLoop
参考：[什么是浏览器的事件循环（Event Loop）？](https://segmentfault.com/a/1190000010622146)
### 1. 准备知识
认识浏览器环境下的异步操作
- 宏任务： `script任务`、`定时器任务`、`ajax`、`用户操作`
- 微任务：`Promise`、`MutationObserver`

说明：一个 Event Loop 只有一个 Microtask Queue，即一轮事件循环结束前会清空整个微任务队列。

### 2. 执行过程

第一轮开始 >>

> 脚本解析（第一个宏任务）：执行完所有同步代码，并初始化（启动）遇到的异步任务，如果有到期的定时器或者其他返回结果的异步任务，将结果和其回调函数放入相应的任务队列。

> 同步函数执行完后，执行栈（ECS）为空，现在开始从微任务队列中取任务放到执行栈中执行，执行任务的过程中，如果遇到新的宏任务或者微任务，则添加到相应的队列。直至微任务队列清空。

<< 第一轮结束
***
第二轮开始 >>

> 从宏任务队列中取一个任务放到执行栈中开始执行，执行的过程中，将遇到的宏任务/微任务放到对应的任务队列。

> 该宏任务执行完后，清空微任务队列

<< 第二轮结束

第三轮，第四轮。。。

### 3. 关于UI渲染
> macro-task任务执行完毕，接着执行完所有的micro-task任务后，此时本轮循环结束，开始执行UI render。UI render完毕之后接着下一轮循环。

**即：MacroTask=>MicroTask=>UI render**

## 二、Node中的EventLoop

思考：为什么同样是JavaScript，node却可以用来服务端编程？

> node是JavaScript的服务端实现，node实现了一些服务端编程需要的api，比如文件操作、net相关、数据库相关，因此具备了服务端编程的能力。

思考：服务端有哪些异步事件？

> 除了客户端JavaScript具有的定时器，用户操作等，还多了io操作

### 1. node提供了4个timer
- setTimeout()
- setInterval()
- setImmediate()
- process.nextTick()

前两个是语言的标准，后两个是node特有的。

### 2. node的事件循环分为6个阶段

> nodejs的event loop分为6个阶段，它们会按照顺序反复运行，分别如下：

```
- timers：执行setTimeout() 和 setInterval()中到期的callback。

- I/O callbacks：上一轮循环中有少数(上一轮执行中超过系统限制的部分)的I/Ocallback会被延迟到这一轮的这一阶段执行，执行一些系统调用错误，比如网络通信的错误回调

- idle, prepare：队列的移动，仅内部使用

- poll：最为重要的阶段，执行I/O callback，在适当的条件下会阻塞在这个阶段

- check：执行setImmediate的callback

- close callbacks：执行close事件的callback，例如socket.on("close",func)
```
不同于浏览器的是，在每个阶段完成后，而不是MacroTask任务完成后，microTask队列就会被执行。这就导致了同样的代码在不同的上下文环境下(浏览器/node)会出现不同的结果。

### 3. 执行过程详解

**注意：所有异步任务的回调都是先从队列取出来，添加到ECS再执行的，也就是说，当执行异步的回调的时候，这个异步任务已经结束了，也不存在于队列中了**

bootstrap阶段 >>

> 执行同步脚本，注册异步任务，完成后清空nextTick队列和microTask队列。（这就是为什么nextTick和微任务一定会比timer先执行的原因）

<< end bootstrap阶段

EventLoop第一轮开始 >>

> 进入timer阶段：检查timer的任务队列（存放**到期未处理**的timer）是否为空，不为空则依次取出执行。

问题： 在执行的过程中，遇到新的异步任务怎么办？

答：有nextTick/或者microTask类型的异步任务，将他们的结果和回调添加至相应队列，然后执行下一个到期未处理的timer回调；如果遇到新的timer，则放到下一轮；如果遇到新的io任务，一般io至少需要10ms才能ok,有可能事件循环已经又过了几轮，因此什么时候执行io的回调并不确定，一般都会晚于同时注册的timer?

> **timer队列清空后**，开始依次清空nextTick队列和microTask队列。然后进入下一个阶段。

举例说明：
```javascript
const fs = require('fs');
const path = require('path');
const timeoutScheduled = Date.now();

setTimeout(() => {
    let delay = Date.now() - timeoutScheduled;

    setTimeout(()=>{
        delay = Date.now() - timeoutScheduled;
        console.log(`${delay}ms have passed since I was scheduled,timer`);
        fs.readFile(path.resolve(__dirname,'4.js'), ()=>{
            console.log('io1')
        });
        setTimeout(()=>{
           console.log('timer1');
        },0);
        process.nextTick(()=>{
            console.log('nextTick1')
        })
        new Promise((resolve,reject)=>{
            resolve();
        }).then(()=>{
            console.log('promise1')
        })
    },0);
    setTimeout(()=>{
        delay = Date.now() - timeoutScheduled;
        console.log(`${delay}ms have passed since I was scheduled,timer`);
         fs.readFile(path.resolve(__dirname,'4.js'), ()=>{
            console.log('io2')
        });
        setTimeout(()=>{
           console.log('timer2');
        },0);
        process.nextTick(()=>{
            console.log('nextTick12')
        })
        new Promise((resolve,reject)=>{
            resolve();
        }).then(()=>{
            console.log('promise2')
        })
    },0);

    // 主线程耗时100ms，确保离开这里时，两个timer都已经过期。
    let start = new Date().getTime();
    while (new Date().getTime() - start < 100) {
    }
    console.log(`${delay}ms have passed since I was scheduled`);
}, 100);

// 说明：
1. 执行完第一行同步代码时，100ms的timer还未到期，因此一直往下来到poll阶段，并阻塞在这里（因为poll队列为空并且immediate队列也为空）。
2. 100ms后，timer到期，跳出poll阶段，到达下一轮循环的timer阶段。
3. timer队列里有两个回调函数，一个一个执行，执行过程中遇到nextTick任务和promise任务都添加到timer阶段的nextTick队列和Promise队列；遇到的timer因为超时至少1ms。
4. 两个回调函数都执行完后，先清空nextTick队列，再清空Promise队列。

// 输出为：
100ms have passed since I was scheduled
202ms have passed since I was scheduled,timer
203ms have passed since I was scheduled,timer
nextTick1
nextTick12
promise1
promise2
timer1
timer2
io1
io2
```


> poll阶段，主要有2个功能：

```
1. 处理 poll 队列的事件
2. 当有已超时的 timer，执行它的回调函数

even loop将同步执行poll队列里的回调，直到队列为空或执行的回调达到系统上限（上限具体多少未详），接下来even loop会去检查有无预设的setImmediate()，分两种情况：

- 若有预设的setImmediate(), event loop将结束poll阶段进入check阶段，并执行check阶段的任务队列

- 若没有预设的setImmediate()，event loop将阻塞在该阶段等待

注意一个细节，没有setImmediate()会导致event loop阻塞在poll阶段，这样之前设置的timer岂不是执行不了了？所以咧，在poll阶段event loop会有一个检查机制，检查timer队列是否为空，如果timer队列非空，event loop就开始下一轮事件循环，即重新进入到timer阶段。
```
<< 第一轮结束
***
第二轮开始 >>

> 从宏任务队列中取一个任务放到执行栈中开始执行，执行的过程中，将遇到的宏任务/微任务放到对应的任务队列。

> 该宏任务执行完后，清空微任务队列

<< 第二轮结束

第三轮，第四轮。。。

以下两个输出的顺序是不固定的，因为Node对timer的过期检查不一定靠谱，它会受机器上其它运行程序影响，或者那个时间点主线程不空闲。

```javascript
setTimeout(() => {
  console.log('timeout')
}, 0)

setImmediate(() => {
  console.log('immediate')
})

// 先输出timeout后输出immediate的原因：
// 初始化脚本+执行同步代码的时间>1ms，因此处理完这些后，timeout已经过期，就在一次事件循环中，先执行timer后执行immediate了

// 先输出immediate后输出timeout的原因：
// 初始化脚本+执行同步代码的时间<1ms，因此处理完这些后，timeout还未过期，就离开timer阶段，执行immediate了。在下一轮事件循环中，执行到期的timer
```


### 解释以下以下问题：
```
// 第二轮
setTimeout(() => console.log(1),0);
setImmediate(() => console.log(2));
// 第一轮
process.nextTick(() => console.log(3));
Promise.resolve().then(() => console.log(4));

(() => console.log(5))();
let start = new Date().getTime();
while(new Date().getTime() - start < 1000){}

// 输出为：
5
3
4
1
2
```
**问题：1000ms以后，所有异步任务都已经有了结果，按照从timer往后的顺序，为什么输出不是：5 1 3 4 2**

> 答案：这要从node的bootstrap阶段到eventloop阶段说起。

参考：
[node源码粗读](https://github.com/xtx1130/blog)

[node源码粗读（9）：nextTick、timers API、MicroTasks注册到执行全阶段解读](https://github.com/xtx1130/blog/issues/20)

![](https://note.youdao.com/yws/api/personal/file/6FD60C834F364AEABA8134EFD0999E06?method=download&shareKey=ce7f7349ca1c11c58b34c215c4fcd9f8)

**1. Environment::Start**

在这里不做过多详细介绍了，之前的文章中介绍过很多。有一个知识点需要注意就是：Immediate是在这个阶段注册到uv_check_start中的。此Immediate其实是Environment::CheckImmediate函数，保证之后event-loop在运行的时候能在check阶段运行这个函数，进而触发immediate_callback_function实现对ImmediateList的调用。
至于idle部分，是为了在有ImmediateList的时候直接跳过poll阶段，毕竟poll是阻塞运行的。

**2. bootstrap阶段**

bootstrap阶段是node运行时候的构建阶段，也是最基础的阶段。bootstrap阶段会把整体的node架子搭起来（在这里不做详细介绍），之后运行业务代码。比如本文中的这个例子，上图中画的应该也比较清晰了，顺序执行。唯一的区别是：在执行不同API的时候，callback的去向是不一致的。从上图中也能看出来，这几个API最终的去向分别是：TimersList、microTask、immediateQueue、nextTickQueue。

其中，setTimeout在注册的时候会创建TimerWrap的实例，在创建实例的时候会初始化uv_timer，之后再通过TimerWrap.start启动uv_timer_start，开始监听，到时间触发运行回调函数：
![](https://github.com/xtx1130/blog/raw/master/images/issue20/issue20-2.png)

==**nextTick在注册之后且bootstrap构建结束后运行SetupNextTick函数，从而触发nextTick的运行，而nextTick在运行之后会触发runMicroTasks()，清空bootstrap阶段的microTask**==：
![](https://github.com/xtx1130/blog/raw/master/images/issue20/issue20-3.png)
> 即：首次运行时，nextTick和microTask是在进入事件循环之前就已经被清空了。

**3. event-loop阶段**

在bootstrap之后便进入了event-loop。event-loop第一个阶段便是timers，在这里如果有到时间的Timer，便会触发OnTimeout，OnTimeout会触发InternalMakeCallback从而执行TimersList中的函数。而在执行完后还会触发InternalCallbackScope::Close，在这个函数中会触发nextTick，在触发nextTick后触发microTasks。==**setTimeout简易流程如下**==：
![](https://github.com/xtx1130/blog/raw/master/images/issue20/issue20-4.png)

也正是InternalMakeCallback和InternalCallbackScope::Close使得libuv和v8紧紧的联系在了一起，一方面可以通过setTimeout来设置运行时间；另一方面又可以在setTimeout的回调中书写js代码。

