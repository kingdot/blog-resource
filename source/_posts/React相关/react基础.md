---
title: react基础
date: 2019-3-20 11:39:01
tag: react
category: JavaScript
---

> ## React仅仅是一个视图层框架，并不具备数据流管理等功能。

> ## React元素是不可变的，创建元素后，您无法更改其子元素或属性。元素就像电影中的单个帧：它代表特定时间点的UI。

## 一、更新UI的方法：

### 1. ReactDOM.render(jsx, el);

大多数ReactApp只调用一次ReactDOM.render方法。

### 2. 调用setState()方法


## 二、函数(无状态)组件的封装

任何一个可以返回jsx元素的函数都被称为一个无状态组件。它们只在初始化时被更新一次。用props.XXX获取传递进来的数据。

## 三、有状态组件的封装

State类似于props，但它是**私有的**并且完全由组件控制。而local state仅适用于用class定义的组件。在class里面使用this.props.XXX获取。

1. 在constructor里面初始化this.state=XXX

2. 在生命周期方法内调用this.setState()更新视图

## 四、组件渲染过程

参考React生命周期图：

![react生命周期](react-life.png)

ReactDOM.render()方法会把传递进去的jsx进行解析，步骤包括：

1. 组件初始化，执行constructor函数，接着调用componentWillMount钩子函数。不建议使用后者。

2. 执行render方法，更新DOM

3. 执行componentDidMount钩子函数

4. 如果其中更改了state或者进行了别的会导致更新DOM的操作，会导致重新render。