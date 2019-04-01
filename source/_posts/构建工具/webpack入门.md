---
title: webpack入门
date: 2019-3-28 11:39:01
tag: webpack
category: JavaScript
---

#### webpack作用

静态资源打包器，输出能在各种浏览器运行的代码，提升了开发至发布过程的效率。

#### 简单配置

配置文件是使用webpack的关键，一份配置文件主要包括：entry，output，mode，loader，plugin

#### 工作原理（基于ES6的模块化的情况下）

> 打包结果

```javascript
(function(modules){
  // ...
})({
  "./src/index.js": (function(){
    // ...
  })
});
```

整个文件只含一个立即执行函数，称为webpackBootstrap，它仅接收一个对象-----未加载的`模块集合(modules)`, 这个modules对象的**key是一个路径，value是一个函数**。

> webpackBootstrap函数体内部

```javascript
// webpackBootstrap
(function(modules){

  // 缓存 __webpack_require__ 函数加载过的模块
  var installedModules = {};

  /**
   * Webpack 加载函数，用来加载 webpack 定义的模块
   * @param {String} moduleId 模块 ID，一般为模块的源码路径，如 "./src/index.js"
   * @returns {Object} exports 导出对象
   */
  function __webpack_require__(moduleId) {
    // ...
  }

  // 在 __webpack_require__ 函数对象上挂载一些变量及函数 ...

  // 传入表达式的值为 "./src/index.js"
  return __webpack_require__(__webpack_require__.s = "./src/index.js");
})(/* modules */);
```

> __webpack_require__函数内部实现

**核心目的：把源文件变成一个module对象，存储module并返回module.exports**

```javascript
function __webpack_require__(moduleId) {
  // 重复加载则利用缓存
  if (installedModules[moduleId]) {
    return installedModules[moduleId].exports;
  }

  // 如果是第一次加载，则初始化模块对象，并缓存
  var module = installedModules[moduleId] = {
    i: moduleId,  // 模块 ID
    l: false,     // 模块加载标识
    exports: {}   // 模块导出对象
  };

  /**
    * 执行模块
    * @param module.exports -- 模块导出对象引用，改变模块包裹函数内部的 this 指向(充当this)
    * @param module -- 当前模块对象引用
    * @param module.exports -- 模块导出对象引用
    * @param __webpack_require__ -- 用于在模块中加载其他模块
    */
  modules[moduleId].call(module.exports, module, module.exports, __webpack_require__);

  // 模块加载标识置为已加载
  module.l = true;

  // 返回当前模块的导出对象引用
  return module.exports;
}
```

> 模块执行函数

\_\_webpack_require__ 中通过 modules[moduleId].call() 运行了模块执行函数，下面我们就进入到 webpackBootstrap 的参数部分，看看模块的执行函数。

```javascript
// webpackBootstrap
(function(modules){
    // ...
})({
    // 入口模块, 参数分别为：当前模块对象，导出对象，模块加载函数
    "./src/index.js": (function(module, __webpack_exports__, __webpack_require__){
        "use strict";
    // 用于区分 ES 模块和其他模块规范，不影响理解 demo，战略跳过。
    __webpack_require__.r(__webpack_exports__);
    
    // 源模块代码中，`import {plus, minus} from './utils/math.js';` 语句被 loader 解析转化。
    
    // 加载 "./src/utils/math.js" 模块，返回的是math.js的exports对象
    /* harmony import */ var _utils_math_js__WEBPACK_IMPORTED_MODULE_0__ = __webpack_require__(/*! ./utils/math.js */ "./src/utils/math.js");

    document.writeln('Hello webpack!');
    // 把源文件中 plus(1, 2) 替换成 mathExports.plus(1, 2);
    document.writeln('1 + 2: ', Object(_utils_math_js__WEBPACK_IMPORTED_MODULE_0__["plus"])(1, 2));
    document.writeln('1 - 2: ', Object(_utils_math_js__WEBPACK_IMPORTED_MODULE_0__["minus"])(1, 2));
    }),
    
     /*** 工具模块 ./src/utils/math.js ***/
    "./src/utils/math.js": (function(module, __webpack_exports__, __webpack_require__) {
    "use strict";

    // 同 "./src/index.js"
    __webpack_require__.r(__webpack_exports__);

    // 源模块代码中，`export` 语句被 loader 解析转化。
    // 导出 __webpack_exports__
    /* harmony export (binding) */ __webpack_require__.d(__webpack_exports__, "plus", function() { return plus; });
    /* harmony export (binding) */ __webpack_require__.d(__webpack_exports__, "minus", function() { return minus; });
    const plus = (a, b) => {
      return a + b;
    };

    const minus = (a, b) => {
      return a - b;
    };
  })
})

```

执行顺序是：入口模块 -> 工具模块 -> 入口模块。入口模块中首先就通过 \_\_webpack_require__("./src/utils/math.js") 拿到了工具模块的 exports 对象。

> 导出模块__webpack_require__.d的实现

**主要目的：把定义为导出的属性挂在exports对象上**

ES 导出语法转化成了\_\_webpack_require__.d(\_\_webpack_exports__, [key], [getter])，而 \_\_webpack_require__.d 函数的定义在 webpackBootstrap 内：

```javascript
// 定义 exports 对象导出的属性。
  __webpack_require__.d = function (exports, name, getter) {

    // 如果 exports （不含原型链上）没有 [name] 属性，定义该属性的 getter。
    // 为什么要用getter（形成闭包），而不是直接使用？？？
    // Object.defineProperty(exports, name, {
    //    enumerable: true,
    //    value: name
    //  });
    // 答案：如果export一个非函数或class，则不会动态绑定
    if (!__webpack_require__.o(exports, name)) {
      Object.defineProperty(exports, name, {
        enumerable: true,
        get: getter
      });
    }
  };

  // 包装 Object.prototype.hasOwnProperty 函数。
  __webpack_require__.o = function (object, property) {
    // 注意：这种调用方式要比直接 obj.hasOwnProperty(key) 好得多，因为不用查找原型链
    return Object.prototype.hasOwnProperty.call(object, property);
  };

// ...
```

可见 \_\_webpack_require__.d 其实就是 Object.defineProperty(obj, key, discriptor) 的简单包装（怪不得叫 d 呢）,区别就是前者的第三个参数不是一个描述对象，而是一个动态返回值得getter函数。




> 本文参考：

[饿了么前端](https://zhuanlan.zhihu.com/p/52826586)     
[从webpack4打包文件说起](https://cloud.tencent.com/developer/article/1172453)
