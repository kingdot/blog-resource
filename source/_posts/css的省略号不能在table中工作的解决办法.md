---
title: css的省略号不能在table中工作的解决办法
date: 2019-1-17 11:27:06
tag: css常用
category: css
---
参考：[https://stackoverflow.com/questions/10372369/why-doesnt-css-ellipsis-work-in-table-cell](https://stackoverflow.com/questions/10372369/why-doesnt-css-ellipsis-work-in-table-cell)

示例：

HTML：
```
<table>
  <tbody>
    <tr><td>Hello Stack Overflow</td></tr>
  </tbody>
</table>
```
CSS：
```
td {
  border: 1px solid black;
  width: 50px;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}
```
JS：
```
$(function() {
  console.log("width = " + $("td").width());
});
```
输出为：width = 139，并且不显示省略号。

##### 原因分析：
> 使用overflow的前提：overflow，width，display，和white-space。

而th、td默认为display: table-cell。

##### 不完美解决方案：
- 更改th、td的display为block
   
    ```
    th，td {
      display: block; /* or inline-block */
    }
    ```
- 对table使用`table-layout: fixed;`。但失去了对列宽的控制。
  
    ```
    table { 
      width: 100%;
      table-layout: fixed; // 表格列宽平均分配容器宽度
    }
    
    td {
      overflow: hidden;
      text-overflow: ellipsis;
      white-space: nowrap;
    }
    ```
##### 最佳解决方案：
```
table {
  width: 100%;
}
// 将你的文本包装成一个<div>并将你的CSS应用到<div>,而不是<td>
th, td > div {
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}
// 此时可以指定列宽,注意!!!  如果使用循环模板，注意给每一个元素都设置同样的大小，调样式时也必须同时更改所有元素宽度才可以！
// th可以不管，他会自适应，只改td就行。
[th,] td { 
  max-width:100px; 
}
```
