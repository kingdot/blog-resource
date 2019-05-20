---
title: 如何监听div的高宽改变
date: 2019-5-20 09:39:01
tag: 监听元素高宽
category: javascript进阶
---

### 已知情况：

- document 类型 dom 有 onresize 事件，在浏览器窗口改变大小时会触发；

- 除了 IE9-10 之外的其他浏览器没有提供给如 div 之类元素的 resize 事件

### 待解决问题

对某个指定元素，比如 div ，通过添加自定义的 onresize 事件，当该 div 的高宽改变时，触发回调函数

### 解决方案

1. 如果只是想在浏览器窗口大小改变时触发，使用 window.onresize 即可

2. IE9-10，默认支持元素的 resize 事件，可以直接通过 div.attachEvent('onresize', handler)实现

2. 使用周期性检查判断元素是否发生了变化（浪费性能，不推荐）

3. 通过 scroll 事件（不直观，不推荐），参考：[巧妙监测元素尺寸变化](https://blog.crimx.com/2017/07/15/element-onresize/)

    具体做法：在被监测元素里包裹一个跟元素位置大小相同的隐藏块。隐藏块可以滚动，并有一个远远大于它的子元素。当被监测元素尺寸变化时期望能触发隐藏块的滚动事件。

    这个方法听起来很简单是不是，但如果你直接这么实现会发现时而行时而不行，问题就在于触发滚动事件的条件。

4. 通过使用 object 元素（给其 type 设置为 ‘text/html’才行）/ iframe 元素（即：document类型元素）来通过 resize 事件监听宿主元素高宽的改变（推荐）, 参考：[JS监听div的resize事件](http://zhangyiheng.com/blog/articles/div_resize.html)

    具体做法：设置object元素的style使其填充满div，这样当div的size发生变化时，object的size也会发生变化。
然后监听object元素的contentDocument.defaultView(window对象)的resize事件。


> 总结：

IE 直接使用原生 resize 事件，其他浏览器通过为宿主元素添加一个同位置等大小且 type 设置为 'text/html' 的 object 元素，监听 object 元素的 resize 事件
