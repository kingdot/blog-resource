---
title: addEventListener和removeEventListener的添加和移除
date: 2019-1-16 11:39:01
tag: JavaScript核心
category: JavaScript
---
## tips
- 回调接受一个参数：一个基于Event 的对象，描述已发生的事件，并且它不返回任何内容。
- 第三个参数是一个布尔值表示是否在捕获阶段调用事件处理程序, 在父子都添加了事件监听的时候，可以决定谁先触发。
- 如果要移除事件句柄，addEventListener() 的执行函数必须使用外部函数，使用匿名函数无法移除

##### addEventListener中的第三个参 数是useCapture, 一个bool类型。当为false时为冒泡获取(由里向外)，true为capture方式(由外向里)。
```
<div id="id1" style="width:200px; height:200px; position:absolute; top:100px; left:100px; background-color:blue; z-index:4">
      <div id="id2" style="width:200px; height:200px; position:absolute; top:20px; left:70px; background-color:green; z-index:1"></div>
  </div>
```

eg1：

```
document.getElementById('id1').addEventListener('click', function() { console.log('id1');}, false);
 
document.getElementById('id2').addEventListener('click', function() { console.log('id2');}, false);
```
点击id2的div结果是: id2， id1
```
document.getElementById('id1').addEventListener('click', function() { console.log('id1');}, false);
 
document.getElementById('id2').addEventListener('click', function() { console.log('id2');}, true);
```
结果是: id2, id1
```
document.getElementById('id1').addEventListener('click', function() { console.log('id1');}, true);
 
document.getElementById('id2').addEventListener('click', function() { console.log('id2');}, false);
```
结果是：id1，id2
```
document.getElementById('id1').addEventListener('click', function() { console.log('id1');}, true);
 
document.getElementById('id2').addEventListener('click', function() { console.log('id2');}, true);
```
结果是：id1，id2

## 测试一
```
var Test = function() {
  this.element = document.body;
  this.handler = function() {
    console.log(this);
  };
  this.element.addEventListener('click', this.handler.bind(this), false);
  this.destroy = function() {
    this.element.removeEventListener('click', this.handler, false);
  };
};
var test = new Test();
```
但是，测试结果发现，调用 test.destroy() 后，点击依旧有效。明明按照以前看的文档说的，**add 和 remove 的时候是同一个函数**啊。

## 测试二
于是，又调整了一下代码。
```
var Test = function() {
  this.element = document.body;
  this.handler = function() {
    console.log(this);
  };
  this.element.addEventListener('click', this.handler, false);
  this.destroy = function() {
    this.element.removeEventListener('click', this.handler, false);
  };
};
```
去掉了 add 时的 bind，再测试发现点击不响应了。

## 结论
经过测试，add 和 remove 事件监听回调时，既**不能使用匿名函数**，也**不能改变指定函数的上下文**，因为bind函数其实会返回一个新的函数。