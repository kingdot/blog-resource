---
title: JavaScript深拷贝、浅拷贝
date: 2019-1-16 11:39:01
tag: JavaScript核心
category: JavaScript
---
参考：[详解javaScript的深拷贝](https://www.cnblogs.com/penghuwan/p/7359026.html)

#### 浅拷贝

核心： 仅仅是值拷贝
```
var a = [1,2,3];
var b = a; // 仅仅是把变量a的内存地址赋给了b，二者指向内存同一块区域
```

#### 深拷贝

- 一层拷贝，适用于单层的js对象或者数组，且值为基本类型变量（如number,String,boolean）。
    - 直接遍历
        ```
        function singleCopy(obj){
            if(typeof obj !== 'object'){
                throw new Error('arguments is valid!');
            }
            
            if(obj.__proto__.constructor === Array){
                var newObj = [];
                for(var item of obj){
                    newObj.push(item);
                }
            } else {
                var newObj = {};
                for(var item in obj){
                    if(obj.hasOwnProperty(item)){
                        newObj[item] = obj[item];    
                    }
                }
            }
            return newObj;
            
        }
        ```
        
    - 对于数组：
        > var newArray = [].slice.call(array_source);
        
        > var newArray = [].concat.call(array_source);
        
    - 对于对象：
        > var newObj = Object.assign({}, obj);

- 深层拷贝
    - 借用JSON对象的方法，简单粗暴，适用除特殊类型（正则，函数）之外的大多数场景。
        > var obj_target = JSON.parse(JSON.stringify(obj_resource))
    
    - 使用递归,可以增加想要适配特殊类型的代码。
        ```
        function deepCopy(obj){
            if(typeof obj !== 'object'){
                throw new Error('params is valid');
            }
            
            var newObj = obj.constructor === Array?[]:{};
            for(var item in obj){
                newObj[item] = typeof obj[item] === 'object' ? deepCopy(obj[item]):obj[item];
            }
            
            return newObj;
        }
        ```