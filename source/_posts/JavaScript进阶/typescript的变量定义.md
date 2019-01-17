---
title: typescript的变量定义
date: 2019-1-16 11:39:01
tag: JavaScript核心
category: JavaScript
---
#### 类变量

##### 面向对象编程时，实例的属性可以通过set方法或者构造器赋值。

```
class A {
    private name: string;
    construtor(name){
        this.name = name;
    }
    
    func1(){
        console.log(this.name); // ??
    }
}
```
类变量有默认值么？


#### 局部变量

##### 务必记住给要使用的对象初始化。

```
let arr:string[];
arr.push('a');
//============  等价于  ===================
let arr;
arr.push('a'); // Cannot read property 'push' of undefined
// 因为使用let声明一个变量后，在没有给他赋值的情况下它的值是undefined。因此也就没有push方法。

let arr:string[] = [];
arr.push('a');
console.log(arr); // ['a']

```

因此要养成初始化变量的习惯。

#### 遍历对象

当对象的key是一个变量时，只能使用obj[key]而不能使用obj.key。当然二者可以合法混用。

```
let obj = {
    person1:{
       name: 'wangdian',
        age: 24,
        sex: 'male' 
    },
    person2:{
       name: 'wangdian',
        age: 24,
        sex: 'male' 
    }
}
for (let key in obj){
//  console.log(obj.key); // undefined
    console.log(obj[key].name);
}
```
