---
title: null、undefined、NaN
date: 2019-1-16 11:39:01
tag: JavaScript核心
category: JavaScript
---
#### Undefined类型
- Undefined类型就只有一个值，即特殊的undefined。在使用var声明变量但未对其加以初始化时，这个变量的值就是undefined。

- 一般不需要显示的将一个变量设置为undefined值的情况，字面值undefined的主要目的是用于比较，区分空对象指针与未经初始化的变量。

- 对于未经声明的变量，使用时会报错，reference Error。

- 声明但未初始化和未声明的变量，使用typeof操作符都会返回undefined。

#### Null类型

- Null类型是第二个只有一个值的数据属性，这个特殊值是null。

- null值表示一个空对象指针，而这也正是使用typeof操作符检测null值时会返回"object"的原因。

- **如果定义的变量要在将来用于保存对象，那么最好将她初始化为null而不是其他值**。这样一来，只要直接检查null值就可以知道相应的变量是否已经保存了一个对象的引用。

- 实际上，undefined值是派生自null值的,因此null==undefined返回true。

#### NaN

- Not a Number，为Number类型的一个属性，代表非数字值的特殊值。
    ```
    语法：Number.NaN
    ```
- ECMAScript能表示的数值区间为：[Number.MIN\_VALUE, Number.MAX_VALUE],超过之后就用一个特殊的Infinity表示。

- 任何涉及NaN的操作(运算)都会返回NaN

- NaN与任何值都不相等，包括它自己

- 判断一个变量是不是number，用**window.isNaN()**函数

    ```
    isNaN(1); // false 
    isNaN('1'); // false
    isNaN("1"); // false
    isNaN('-0.2'); // false
    isNaN('-0.2a'); // true
    isNaN("a"); // true
    isNaN(false); // false
    isNaN(true); // false
    isNaN(null); // false
    isNaN(undefined); // true
    
    isNaN(new Array(1,2)); // true
    ```
    - isNaN()在接收到一个值之后，会尝试将这个值转换为数值。某些不是数值的值会直接转换为数值，例如字符串"10"或 Boolean 值。而任何不能被转换为数值的值都会导致这个函数返回true。
   
    - isNaN()也可以接受一个对象作为参数，它会首先调用对象的valueOf方法，然后确定该方法返回的值是否可以转换为数值，如果不能，则基于这个返回值再调用toString方法，再测试返回值。
    

#### null和undefined异同

- 二者都是5种基本类型之一，但并不像其他三种(String、Number、Boolean)拥有构造函数，因此不是构造器类型，无法使用new，也无法拥有任何属性和方法。

- null是关键字；undefined是Global对象的一个属性

- null是对象(空对象, 没有任何属性和方法)；undefined是Undefined类型的值。试试下面的代码： 
   ```
   document.writeln(typeof null); //return object 
   document.writeln(typeof undefined); //return undefined 
   ```
- 对象模型中，所有的对象都是Object或其子类的实例，但null对象例外： 
    ```
    document.writeln(null instanceof Object); //return false 
    ```
- null等值(\==)于undefined，但不全等值(===)于undefined： 
    ```
    document.writeln(null == undefined); //return true 
    document.writeln(null === undefined); //return false 
    ```
- 运算时null与undefined都可以被类型转换为false，但不等值于false： 
    ```
    document.writeln(!null, !undefined); //return true,true 
    document.writeln(null==false); //return false 
    document.writeln(undefined==false); //return false 
    ```

