---
title: 关于数组的forEach和map方法
date: 2019-1-16 11:39:01
tag: JavaScript核心
category: JavaScript
---
### 错误的理解
```
var bb = [1,2,3,5];
bb.forEach((item)=>{item = 3}); 
bb; // [1, 2, 3, 5]

var bb = [{key:1},{key:2}];
bb.forEach((item)=>{item.key =3});
bb; // [{key: 3},{key: 3}]
```
因为forEach和map一样，返回的是一个新的数组。

#### 错解分析：
因为item是数组元素的一个拷贝，类似函数传参。当item为基本类型时，拷贝的是副本，当为复杂类型时为引用。
***
### 正确的理解

原文: [JavaScript — Map vs. ForEach - What’s the difference between Map and ForEach in JavaScript?](https://blog.fundebug.com/2018/02/05/map_vs_foreach/)

如果你已经有使用JavaScript的经验，你可能已经知道这两个看似相同的方法：`Array.prototype.map()`和`Array.prototype.forEach()`。

那么，它们到底有什么区别呢？

#### 定义
我们首先来看一看MDN上对Map和ForEach的定义：

- `forEach()`: 针对每一个元素执行提供的函数(executes a provided function once for each array element)。
- `map()`: 创建一个新的数组，其中每一个元素由调用数组中的每一个元素执行提供的函数得来(creates a new array with the results of calling a provided function on every element in the calling array)。

#### 到底有什么区别呢？
**forEach()方法不会返回执行结果，而是undefined**。也就是说，forEach()会修改原来的数组，而**map()方法会得到一个新的数组并返回**。

##### 关于forEach(item,index,arr)会修改原来数组的解释：
- 当item为基本类型时：通过arr[index]修改
- 当item为复杂类型时：直接修改item即可

#### 示例
下方提供了一个数组，如果我们想将其中的每一个元素翻倍，我们可以使用map和forEach来达到目的。
```javascript
let arr = [1, 2, 3, 4, 5];
// ForEach:
arr.forEach((num, index) => {
    return arr[index] = num * 2;
});
//执行结果如下：
// arr = [2, 4, 6, 8, 10]
// Map:
let doubled = arr.map(num => {
    return num * 2;
});
//执行结果如下：
// doubled = [2, 4, 6, 8, 10]
```
#### 执行速度对比
可以看到，在我到电脑上forEach()的执行速度比map()慢了70%。

#### 函数式角度的理解
如果你习惯使用函数是编程，那么肯定喜欢使用map()。因为forEach()会改变原始的数组的值，而map()会返回一个全新的数组，原本的数组不受到影响。

#### 哪个更好呢？
取决于你想要做什么。

- **forEach适合于你并不打算改变数据的时候，而只是想用数据做一些事情 – 比如存入数据库或则打印出来**。
```javascript
let arr = ['a', 'b', 'c', 'd'];
arr.forEach((letter) => {
    console.log(letter);
});
// a
// b
// c
// d
```
- **map()适用于你要改变数据值的时候。不仅仅在于它更快，而且返回一个新的数组。这样的优点在于你可以使用复合(composition)(map(), filter(), reduce()等组合使用)来玩出更多的花样**。
```javascript
// 我们首先使用map将每一个元素乘以2，然后紧接着筛选出那些大于5的元素。最终结果赋值给arr2。

let arr = [1, 2, 3, 4, 5];
let arr2 = arr.map(num => num * 2).filter(num => num > 5);
// arr2 = [6, 8, 10]
```

#### 核心要点
- 能用forEach()做到的，map()同样可以。反过来也是如此。
- map()会分配内存空间存储新数组并返回，forEach()不会返回数据。
- forEach()允许callback更改原始数组的元素。map()返回新的数组。