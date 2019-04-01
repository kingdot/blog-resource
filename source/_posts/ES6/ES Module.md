---
title: ES Module
date: 2019-1-17 11:45:56
tag: ES6，ES7新特性
category: ES6
---
#### import {***} from "abcd";

表示从最近的node_modules里面找

#### import {***} from "./abcd";

表示按相对路径找

#### export 后面跟着的语句需要具有`声明部分`和`赋值部分`。

1. 声明部分为export提供了所暴露接口的标识

2. 赋值部分为export提供了接口的值

```
export const apiRoot = 'fsalfjslfj';

export function method(){}

export class Foo{}
```

3. 默认导出不需要声明部分

```
export default <value>
```