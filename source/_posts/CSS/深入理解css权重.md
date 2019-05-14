---
title: 深入理解css权重
date: 2019-5-14 21:39:01
tag: css权重
category: css
---

参考：[https://juejin.im/post/5afaa228f265da0b8070e41d#heading-11](https://juejin.im/post/5afaa228f265da0b8070e41d#heading-11)

|    元素    | 贡献值 |
| :--------: | :---: |
| 继承、*    |  0，0，0，0 |
| 每个标签   |  0，0，0，1 |
| 类、伪类   |  0，0，1，0 |
| ID   | 1，0，0，0 |
| 行内样式   |  1，0，0，0 |
| !important   |  无穷大 |
| max-height、max-width覆盖width、height   |  大于无穷大 |
| min-height、min-width   |  大于max-height、max-width |

> 总结

计算规则


遇到有贡献值的就进行累加，例如：

```
div ul li ---> 0,0,0,3
.nav ul li ---> 0,0,1,2
a:hover ---> 0,0,1,1
.nav a ---> 0,0,1,1
#nav p ---> 0,1,0,1
```

数位没有进位：

0,0,0,5+0,0,0,5 = 0,0,0,10而不是0,0,1,0，所以不会存在10个div能赶上一个类选择器的情况 。

权重不会继承，所以父元素的权重再高也和子元素没有关系

如果有!important那么相加的那些无论多高就不管用，

如果有max-height/max-width那么!important不管用，

如果同时有min-height/min-width和 max-height/max-width，那么max-height/max-width的不管用。