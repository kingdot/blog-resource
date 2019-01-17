---
title: css3渐变
date: 2019-1-17 11:27:06
tag: css常用
category: css
---
> 本文整理自MDN
#### 渐变适用范围
- CSS linear-gradient() 函数用于**创建一个表示两种或多种颜色线性渐变的图片**。其结果属于<gradient>数据类型，是**一种特别的<image>数据类型**。

- 由于<gradient>数据类型系<image>的子数据类型，**<gradient>只能被用于<image>可以使用的地方**。因此，linear-gradient() 并不适用于background-color以及类似的使用 <color>数据类型的属性中。

#### 渐变种类

> 线性渐变 linear-gradient()

> 径向渐变 radial-gradient()

> 重复渐变 repeating-linear-gradient()、 repeating-radial-gradient()

#### 渐变使用方法

1. [线性渐变](https://developer.mozilla.org/zh-CN/docs/Web/CSS/linear-gradient)
    - 线性渐变相关概念：
        - *渐变线*：渐变线由**包含渐变图形的容器的中心点**和**一个角度**来定义的。渐变线上的颜色值是由不同的点来定义，包括起始点，终点，以及两者之间的可选的中间点（中间点可以有多个）。 
        - *起始点*：起始点是渐变线上代表起始颜色值的点。起始点由渐变线和过容器顶点的垂直线之间的交叉点来定义。（垂直线跟渐变线在同一象限内）
        - *终点*：终点是渐变线上代表最终颜色值的点。终点也是由渐变线和从最近的顶点发出的垂直线之间的交叉点定义的，然而从起始点的对称点来定义终点是更容易理解的一种方式，因为终点是起点关于容器的中心点的反射点。
        - *线性渐变的构成*：渐变线轴上的每个点都具有独立的颜色。为了构建出平滑的渐变，**linear-gradient() 函数构建一系列垂直于渐变线的着色线，每一条着色线的颜色则取决于与之垂直相交的渐变线上的色点**
    - 简单线性渐变(最终语法)：
        > linear-gradient([ [ [ <angle> | to [top | bottom] || [left | right] ],]? <color-stop>[, <color-stop>]+);
         
        > 解释一下：角度和to ×× 可选且只选其一，颜色终点必选且允许有多个；颜色终点可选跟一个表示距离的指定长度或者百分比，形式为：color1 距离1，color2 距离2；距离指的是color所在渐变线上的点与起始点的距离（不是坐标）。

        > 参考：[深入理解线性渐变](https://www.zhangxinxu.com/wordpress/2013/09/%E6%B7%B1%E5%85%A5%E7%90%86%E8%A7%A3css3-gradient%E6%96%9C%E5%90%91%E7%BA%BF%E6%80%A7%E6%B8%90%E5%8F%98/)

    - 关于角度的使用：
        - 带有浏览器前缀的（-webkit和-moz）：使用极坐标定义<angle>参数，导致了0deg指向东方，且顺时针方向角度增加。
        - 不带有前缀的，W3C标准：与CSS的其他部分保持一致，将0deg指向北方，且顺时针方向角度增加。
        - chrome下，缺省角度时，默认为180deg，即从上到下。
        - 如果指定的前后两个color-stop与渐变线方向相反，即后者距离小于前者时，后者的距离失效，且整体失去渐变效果。
        - 如果两个color-stop构成的区间α位于[start,end]之间，则在α区间内呈现渐变效果，在α之外保持与临近端点相同颜色。

2. [径向渐变](https://developer.mozilla.org/zh-CN/docs/Web/CSS/radial-gradient)--待整理