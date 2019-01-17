---
title: ES6 新特性
date: 2019-1-17 11:45:56
tag: ES6，ES7新特性
category: ES6
---
#### 1. 新增let、const命令

- let：声明一个变量
    - 不会变量提升，存在暂时性死区，因此以前声明在{... ...}里面的变量改用let声明后并不会提升到函数作用域或者全局作用域，实际上也就形成了**块级作用域**。
    - 有了块级作用域后，广泛使用的立即执行函数(function(){... ...})();已经不需要了。
        > 暂时性死区的本质: 只要一进入当前作用域，所要使用的变量就已经存在了，但是不可获取，只有等到声明变量的
那一行代码出现，才可以获取和使用该变量。

    - ES5规定只能在全局或者函数环境下声明函数。ES6中，在块级作用域里也可以声明的函数。只是函数声明语句类似于let，不能在块级作用域外使用。
    - 但是在**支持ES6的浏览器**中，在**块级作用域中声明函数**类似于var，即**会提升到全局作用域或者函数作用域的头部**。同时函数声明**还会提升到所在的块级作用域的头部**。
    - 因使用环境差异导致块级作用域解析不一致，所以应尽量避免在块级作用域中声明函数，如果确实需要，应写成函数声明表达式。
- const：声明一个常量
    - 一旦声明，就必须初始化，且声明后常量的值就不能变。
    - 作用域问题同let
    - const 实际上保证的，并不是变量的值不得改动，而是变量指向的那个内存地址不得改动。

#### 2. ES6声明变量的6种方法：

- 在ES5中，声明变量只有var和function两种方法
- ES6中，增加了let、const、import、class四种

#### 3. 在全局环境使用let和const声明的变量不再是全局对象的属性，仅仅只是全局变量。

#### 4. 数组和对象的解构
##### 本质：属于模式匹配
##### 使用场景：主要用在变量赋值和函数传参中

- 数组：只要某种数据具备Iterator接口，都可以使用数组的解构。
    ```
    let [a,b,c] = [1,2,3];
    let [,,c] = [1,2,3];
    let [a,b,...c] = [1,2,3,4,5];  // c: [3,4,5]
    
    解构不成功的值为undefined，展开操作符得到的是空数组
    let [a,b,c,...d] = [1];  // b,c:undefined  d:[]
    
    let [x, y, z] = new Set(['a', 'b', 'c']);
    
    数组解构允许指定默认值
    let [foo = true] = []; // foo:true
    let [b,c = 2] = [1]; // c:2
    
    判断一个位置是否有值，是使用(XXX === undefined)的：
    let [x = 1] = [undefined];  // x = 1
    let [x = 1] = [null];  //x = null
    ```
    
- 对象；对象的解构与数组有一个重要的不同。数组的元素是按次序排列的，变量的取值由它的位置决定；而对象的属性没有次序，变
量必须与属性同名，才能取到正确的值。
    ```
    let { foo, bar } = { foo: "aaa", bar: "bbb" };   // foo: "aaa"    bar: "bbb"
    
    let { foo: baz } = { foo: "aaa", bar: "bbb" };  
    // baz: "aaa"
    // foo: error: foo is not defined
    // 也就是说，对象的解构赋值的内部机制，是先找到同名属性，然后再赋给对应的变量。真正被赋值的是后者，而不是前者。
    
    注意，采用这种写法时，变量的声明和赋值是一体的。对于 let 和 const 来说，变量不能重新声明，所以一旦赋值的变量以前声明过，就会报错。
    let foo;
    let {foo} = {foo: 1}; // SyntaxError: Duplicate declaration "foo"
    let baz;
    let {bar: baz} = {bar: 1}; // SyntaxError: Duplicate declaration "baz
    ```
#### 5. 展开运算符
##### 1. 当展开运算符(...args)出现在函数的最后一个参数（也可以是唯一一个）时，起到合并实参到一个名为args的数组中的作用。
##### 2. 其他情况：

- 数组的展开：
    > let arr1 = [1,2,3],arr2 = [4,5,6];  let arr3 = [...arr1,...arr2];
    > let arr3 = [...[1,2,3], ...[4,5,6]];
- 对象的展开：
    > let obj1 = {key1: 1}, obj2 = {key2:2}; let obj3 = {...obj1, ...obj2};

#### 6. Promise
- Promise是一种异步流程控制手段。

- Promise的构造函数接收一个函数（Excutor）作为参数
- Excutor接收两个函数，resolve和reject，可以在Excutor体内主动调用。
- new Promise之后，Excutor函数会立即执行
- 每一个promise的实例上都会有一个then方法，有两个参数，一个成功的回调，一个失败的回调
- then中如果返回的是promise，会取这个promise的结果，传递到下一个then里面。如果返回的是值，这个值会直接传递到下一个then的onfulfilled。如果没有显式return，则给下一个then的onfulfilled传递undefined。
- then中如果返回promise，并不是返回了this，而是返回了一个新的promise。
- 可以使用catch统一处理来自Excutor或者来自then的错误和异常。
- Promise.all([prom1,prom2]).then(([res1,res2])=>{})，并发发出多个异步请求，只有在所有都成功时，then才成功，有任何一个失败，都会走then的失败。
- Promise.race([])，并发发出多个请求，第一个成功或者失败的会进入then。
- 解决回调地狱的方法：在一个then里面返回一个新的promise，就可以把嵌套的promise改成链式调用的。