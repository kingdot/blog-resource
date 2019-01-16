---
title: this相关问题
date: 2019-1-16 11:39:01
tag: JavaScript核心
category: JavaScript
---
#### 对this的概括：

1. this是执行上下文（Execution Context）的一个重要属性，是一个与执行上下文相关的特殊对象。因此，它可以叫作**上下文对象**（也就是用来指明执行上下文是在哪个上下文中被触发的对象）。

2. **this不是变量对象（Variable Object）的一个属性**，所以跟变量不同，**this从不会参与到标识符解析过程**。也就是说，在代码中当访问this的时候，它的值是直接从执行上下文中获取的，**并不需要任何作用域链查找**。**this的值只在进入上下文的时候进行一次确定。**

#### this的确定方法：

- *全局上下文中*：无论是否在严格模式下，在全局执行上下文中（**在任何函数体外部**）this 都指代全局对象。

- *函数上下文*：**在function内部出现this**，this的值取决于函数被调用的方式。
    - function作为对象的方法调用：指向调用的对象
    - function作为构造函数：指向new出的对象
    - function作为无上下文的函数：也就是说**这些函数没有绑定到特定的对象上**，那么这些上下文无关的函数将会被默认的绑定到global object上。比如：**闭包和立即执行函数**。
- 作为eventhaddler时：指向触发事件的页面元素对象

#### this的问题解决

- var that = this；
- 函数的bind方法
- 箭头函数

#### 函数作为参数时，参数函数里面的this指向？

当函数作为参数时，这个函数便失去了所属对象信息，单纯作为一个函数。因此，在调用者内部执行该函数的时候，该函数里面的this指向global对象。

#### 箭头函数的this

- 箭头函数执行时，自身（本函数）没有this，因此如果使用this的话会当作普通变量从作用域查找。
- 作为对比，普通function函数执行时，由于其执行上下文中必定有this，因此永远不会去作用域链查找this。因此，this值也就取决于执行上下文建立的那一刻，也就是当函数被调用，但是开始执行函数内部代码之前。
    ```
    补充：上下文创建和代码执行阶段的EC
     
     // demo
     function foo(i) {
        var a = 'hello';
        var b = function privateB() {};
        function c() {}
    }

    foo(22);

     // 创建EC，代码未执行
     fooExecutionContext = {
        scopeChain: { ... },
        variableObject: {
            arguments: {
                0: 22,
                length: 1
            },
            i: 22,
            c: pointer to function c()
            a: undefined,
            b: undefined
        },
        this: { ... }
    }
    
    // 代码开始执行时
    fooExecutionContext = {
        scopeChain: { ... },
        variableObject: {
            arguments: {
                0: 22,
                length: 1
            },
            i: 22,
            c: pointer to function c()
            a: 'hello',
            b: pointer to function privateB()
        },
        this: { ... }
    }
    ```


- 对象里面没有this：由于this是函数的执行上下文中的变量，因此只有在函数内才有this。
    ```
    var obj = {
        a: ()=>{console.log(this);}
    };
    obj.a(); // global object
    
    var obj2 = {
        a: function(){
            console.log(this);
            (()=>{console.log(this);})();
        }
    };
    obj2.a(); // obj2, obj2
    
    
    ```
