---
title: reduce
date: 2019-1-16 11:39:01
tag: JavaScript基础
category: JavaScript
---
#### `reduce`为ES6中新增的方法，接受两个参数，`回调函数`和`初始值`。

#### 回调函数接受4个参数，除了与map、forEach等相同的后三个参数外，还多了第一个累计值参数

#### 初始值会在第一次调用回调函数时作为第一个参数传入

#### 以后每次循环的累计值，是根据上次回调函数return的结果定义的，如果在回调函数体中不改变或者不return，累计值不会改变。

#### 最终reduce函数返回的是一个值