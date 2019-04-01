---
title: MessageChannel用法
date: 2019-3-18 11:39:01
tag: JavaScript进阶
category: JavaScript
---
#### 使用方法

> let channel = new MessageChannel();

#### 实例属性

> channel.port1  
channel.port2

#### port1/2的方法

> port1.postMessage()  
port1.onmessage(e=>{e.data})

简单来说，MessageChannel创建了一个通信的管道，这个管道有两个端口，每个端口都可以通过postMessage发送数据，而一个端口只要绑定了onmessage回调方法，就可以接收从另一个端口传过来的数据。

#### 举个例子

```javascript
let ch = new MessageChannel();
const {port1, port2} = ch;

port1.onmessage = (e)=>console.log('port1 data:', e.data);
port2.onmessage = (e)=>console.log('port2 data:', e.data);

port1.postMessage('i am port1');
port2.postMessage('i am port2');
```

#### webWorker是一个独立的线程，它可以通过实例方法postMessage、onmessage和主线程页面通信。

#### 那么，有没有办法让多个webWorker互相通信呢？

答案是肯定的，我们可以通过worker的postMessage实例方法将MessageChannel实例的port传递过去，通过port通信

```javascript
// main.js
let w1 = new Worker("worker1.js");
let w2 = new Worker("worker2.js");

let ch = new MessageChannel();
const {port1,port2} = ch;

// 错误的方法: 因为port是只读的，不能被深拷贝过去用于通信
w1.postMessage(port1);

// 正确的做法：
w1.postMessage(someObj, [port1]); // 第二个参数用于转移对象所有权，被转移后，在发送它的上下文中将变得不可用，并且只有在它被发送到的worker中可用。 
w2.postMessage(someObj, [port2]); 
w2.onmessage(e=>console.log(e.data)); // 观察worker2收到的数据

// worker1.js
self.onmessage = (e)=>{
    const  port1 = e.ports[0];
    port.postMessage('send to worker2');
}

// worker2.js
self.onmessage = (e)=>{
    const  port2 = e.ports[0];
    port2.onmessage(ms=>{
        self.postMessage(ms); // 因为worker线程不能使用console.log，因此需要把数据传回主线程打印观察
    });
}

```

总结：消息传递路径为：worker1 => port1 => port2 => worker2