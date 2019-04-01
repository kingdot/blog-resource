---
title: npm基础
date: 2019-3-16 11:39:01
tag: npm基础
category: JavaScript
---

### `package.json`和`package-lock.json`

> package.json

#### 1. 主要作用：

记录`npm install`安装的包信息，定义一些工程描述信息等。

#### 2. 常用字段意义说明

```javascript
{
  "name": "blog",  //项目名称
  "version": "0.0.0",   //版本
  "description": "",   //项目描述
  "private": true,  
  "main": "index.js",  //入口文件
  "scripts": {   //配置一些通用的命令脚本
    "start": "node ./bin/www"
  },
  "keywords": [],  //项目的关键字
  "author": "",  //作者
  "dependencies": {   //开发时的依赖
    "body-parser": "~1.16.0"
  },
  "devDependencies": {   //运行时的依赖
    "express-session": "^1.15.1"
  }
}

注：dependencies和devDependencies的区别在于，当执行npm install --production时，只会安装dependencies定义的依赖
```

#### 3. 关于版本号规则

版本格式： major.minor.patch => 主版本.次版本.修订版本

> `^` 当指定minor时，minor不变，patch可向上增加；当仅指定major时，minor和patch可任意

```
如：~1.1.2，表示>=1.1.2 <1.2.0，可以是1.1.2，1.1.3，1.1.4，.....，1.1.n 

如：~1.1，表示>=1.1.0 <1.2.0，可以是同上

如：~1，表示>=1.0.0 <2.0.0，可以是1.0.0，1.0.1，1.0.2，.....，1.0.n，1.1.n，1.2.n，.....，1.n.n
```

> `~` 兼容某个版本，版本号中最左边的非0数字的右侧可以任意。如果缺少某个版本号，则这个版本号的位置可以任意

```
^1.2.3 := >=1.2.3 <2.0.0

^0.2.3 := >=0.2.3 <0.3.0

^0.0.3 := >=0.0.3 <0.0.4
```

#### 4. 常用命令

```
npm init -y(--yes)   // 初始化package.json

npm install (-g) (--save-dev / -D  --save/-S)  // 本地/全局安装

npm uninstall （-g）packageName  // 卸载本地/全局包

npm root // 查看本地包位置

npm root -g  // 查看全局包位置

npm update （-g） // 更新模块

npm ls (-g) // 查看当前安装的模块（包）

npm info  //查看模块（包）的信息：

npm cache verify // 清楚缓存

```

#### 5. 切换npm源

```
//1.先安装nrm工具
$ npm install -g nrm

//2.查看当前可用的镜像源
$ nrm ls

//3.切换npm源
$ nrm use 镜像源名称
```

*********************************************

> package-lock.json

```json
{
  "name": "china-telecom",
  "version": "1.0.0",
  "lockfileVersion": 1,
  "requires": true,
  "dependencies": {
      "set-value": {
          "version": "2.0.0",
          "resolved": "https://registry.npmjs.org/set-value/-/set-value-2.0.0.tgz",
          "integrity": "sha512-hw0yxk9GT/Hr5yJEYnHNKYXkIA8mVJgd9ditYZCe16ZczcaELYYcfvaXesNACk2O8O0nTiPQcQhGUQj8JLzeeg==",
          "dev": true,
          "requires": {
            "extend-shallow": "^2.0.1",
            "is-extendable": "^0.1.1",
            "is-plain-object": "^2.0.3",
            "split-string": "^3.0.1"
          },
          "dependencies": {
            "extend-shallow": {
              "version": "2.0.1",
              "resolved": "https://registry.npmjs.org/extend-shallow/-/extend-shallow-2.0.1.tgz",
              "integrity": "sha1-Ua99YUrZqfYQ6huvu5idaxxWiQ8=",
              "dev": true,
              "requires": {
                "is-extendable": "^0.1.0"
              }
            }
          }
        }
    }
```    
    
// 注意：requires和dependencies的区别：

假设包X依赖于Z1.0版中的包，而包Y依赖于相同的包Z，但是在2.0版中。在这种情况下，我们必须安装此软件包的两个版本。一个将安装在根node_modules文件夹中，另一个将安装在node_modules依赖包的文件夹中，例如

node_modules
   /X
   /Y
      /Z@2.0
   /Z@1.0
   
有了这些知识，它很容易理解：

“requires”反映了'package.json'文件中的依赖项，而“dependencies”反映了此依赖项的node_modules文件夹中实际安装的依赖项。

默认情况下，所有依赖项都安装在root node_modules中，但如果存在冲突，则它们将安装在该特定依赖项的node_modules中。

package.json里面定义的是版本范围（比如^1.0.0），具体跑npm install的时候安的什么版本，要解析后才能决定，这里面定义的依赖关系树，可以称之为逻辑树（logical tree）。

node_modules文件夹下才是npm实际安装的确定版本的东西，这里面的文件夹结构我们可以称之为物理树（physical tree）。

安装过程中有一些去重算法，所以你会发现逻辑树结构和物理树结构不完全一样。

package-lock.json可以理解成对结合了逻辑树和物理树的一个快照（snapshot），里面有明确的各依赖版本号，实际安装的结构，也有逻辑树的结构。其最大的好处就是能获得可重复的构建（repeatable build），当你在CI（持续集成）上重复build的时候，得到的artifact是一样的，因为依赖的版本都被锁住了。