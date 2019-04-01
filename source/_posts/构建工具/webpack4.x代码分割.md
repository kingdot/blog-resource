---
title: webpack4.x代码分割
date: 2019-3-16 11:39:01
tag: webpack
category: JavaScript
---

> 为什么要代码分割？

它允许将代码拆分成多个捆绑的包，然后对这些包进行按需加载或者并行加载，使用得当可以达到首次加载速度的提成。

> 三种实现代码分割的通用方法，可以搭配使用

```
1. 入口文件：通过手动配置多个webpack入口文件

2. 避免重复: 使用splitChunks插件删除并提取出公共代码作为一个单独的chunk

3. 动态导入：调用模块内的内联函数如import()或者require.ensure()异步加载代码
```

#### 一、入口文件

> 最简单、最直观的代码分割方式，但缺陷也很明显

```
// ...

entry: {  //配置多个入口文件打包成多个代码块
        index: path.resolve(root_path, 'index.js'), 
        a: path.resolve(root_path, 'a.js')
    },
    
// ...
```

这样使得无关系的两个js模块可以并行加载，这种方式相比较于加载一个等价大小的js模块有速度上的提升。

但是也存在以下问题

1.如果index.js也引入了lodash，那么lodash将会同时打包到两个js模块中

2.不灵活，不能根据核心应用程序的逻辑来动态分割代码

#### 二、避免重复

> splitChunk插件（webpack4.x之前使用CommonsChunkPlugin）允许我们将公共依赖项提取到现有的entry chunk或全新的代码块中

适用场景：两个或多个独立的模块都使用了相同的第三方类库，此时就可以把该类库提取出来，这样，只下载一次就可以了。

`splitChunk`插件默认配置：

```
1. 模块被重复引用或者来自node_modules中的模块

2. 在压缩前最小为30kb（即大于30kb才分割）

3. 在按需加载时，请求数量小于等于5 （小于5才分割）

4. 在初始化加载时，请求数量小于等于3 （小于3才分割）
```
webpack使用SplitChunk插件：

```javascript
optimization: {
     splitChunks: {
         chunks: 'all'
     }
}
```

注意：`xtract-text-webpack-plugin` 已经被 `mini-css-extract-plugin`替代

#### 三、动态导入

webpack会自动分割动态导入的代码。

有两种方式供我们实现动态导入：

```
1. import() // ES的提案，返回一个promise，导入的模块在then中拿到

2. require.ensure() // webpack在编译时会静态的解析代码中的require.ensure(),将里面
require的模块添加到一个分开的chunk中，这个新chunk会被webpack通过jsonp来按需加载。
```

下面主要使用import()实现动态导入，因为import()目前为止还是提案阶段，需要安装插件npm install babel-plugin-syntax-dynamic-import --save-dev，.babelrc配置如下

```
{
    "presets": [
        ...
    ],
    "plugins": ["syntax-dynamic-import"]
}
```
下面看核心代码

webpack.config.js：
```
var path = require('path');
var root_path = path.resolve(__dirname);
module.exports = {
    mode: 'production',
    entry: {
        index: path.resolve(root_path, 'index.js'),
    },
    output: {
        filename: '[name].bundle.js',
        chunkFilename: '[name].demand.js',
        path: path.resolve(root_path, 'dist')
    },
    module: {
        rules: [
            {
                test: /\.js$/,
                loader: 'babel-loader',
                exclude: path.resolve(root_path, 'node_modules'),
                include: root_path
            }
        ]
    }
}
```

index.js：点击按钮才异步加载lodash

```
let btn = document.getElementById('btn');

btn.addEventListener('click', () => {
    import(/* webpackChunkName: "lodash" */ 'lodash').then(
        _ => {
            var app = document.getElementById('app');
            app.textContent = _.join(['Index', 'Module', 'Loaded!'], ' ');
        }
    ).catch(
        err => {
            console.log('loading module error occur', err);
        }
    )
});
```

`chunkFilename`：采用注释的写法指定非entry chunk的文件名，比如import()引入的模块不打包进entry中，而会作为单独的chunk打包，其文件名就由该属性决定

链接：[https://www.jianshu.com/p/3c93c0e724ba](https://www.jianshu.com/p/3c93c0e724ba)。