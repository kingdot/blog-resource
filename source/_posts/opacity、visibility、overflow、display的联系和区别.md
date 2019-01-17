---
title: opacity、visibility、overflow、display的联系和区别
date: 2019-1-17 11:27:06
tag: css常用
category: css
---
## 区别和作用

``` 
    1. display:none

    该方式让元素隐藏时，隐藏的元素不占空间，隐藏后将改变html原有样式。即会导致回流和重绘，降低性能。

    2. visibility:hidden

    该方式让元素隐藏时，隐藏的元素还是占用原有位置，隐藏后不将改变html原有样式。但，如果该元素的子元素使用了visibility:visible的话，改子元素将不被隐藏。

    3. opacity:0

     该方式让元素隐藏时，隐藏的元素还是占用原有位置，隐藏后不将改变html原有样式。但，隐藏的元素所对应的事件，仍然可以触发。
     
    4. overflow: hidden
    
    该方式把超出容器的部分全部切掉，即超出的部分不仅不显示，而且也不占据空间。
    
    通过overflow和height也能达到彻底隐藏一个元素的目的
        
        height：0；    
        overflow：hidden；
    
```