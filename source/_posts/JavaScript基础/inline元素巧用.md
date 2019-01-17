---
title: inline元素巧用
date: 2019-1-17 11:27:06
tag: css常用
category: css
---
## inline、block、inline-block的区别

### inline: 

- inline元素不会独占一行，多个相邻的行内元素会排列在同一行里，直到一行排列不下，才会新换一行，其宽度随元素的内容而变化。

- inline元素设置width,height属性无效。

- inline元素的margin和padding属性，水平方向的padding-left, padding-right, margin-left, margin-right都产生边距效果；但竖直方向的padding-top, padding-bottom, margin-top, margin-bottom不会产生边距效果。

### block: 

- block元素会独占一行，多个block元素会各自新起一行。默认情况下，block元素宽度自动填满其父元素宽度。

- block元素可以设置width,height属性。块级元素即使设置了宽度,仍然是独占一行。

- block元素可以设置margin和padding属性。

### inline-block：

- 简单来说就是将对象呈现为inline对象，但是对象的内容作为block对象呈现。之后的内联对象会被排列在同一行内。比如我们可以给一个link（a元素）inline-block属性值，使其既具有block的宽度高度特性又具有inline的同行特性。

## inline-block元素的巧用

1. 在使用bootstrap的网格系统时，当给一个块元素设置了小于100%的width后，会给父容器(col-x-x)右侧留下一段空间，当然，我们可以使用`margin: 0 auto;`来让这个不满width的block居中，同时还可以通过对该block设置`display: inline-block`,从而达到自动缩小该block的宽度

2. 巧用inline-block做列表布局 

    参考：[巧用inline-block做列表布局](https://www.jianshu.com/p/9fb9697832a0)

    ```HTML
    <ul>
        <li>someBlock</li>
        <li>someBlock</li>
        <li>someBlock</li>
        <li class="fix">someBlock</li>
    </ul>
    ```

    ```SCSS
    ul{
        padding: 20px;
        font-size: 0;
        
        li{
            display: inline-block;
            width: 100px;
            padding: 0 10px;
            vertical-align: top; // 顶端对齐
            text-align: justify; // 段落文本（行元素）两端对齐
            list-style: none;
            font-size: 16px;
            box-sizing: border-box;
        }
    }
    ```

    注意：列表（或文字）要两端对齐的前提就是内容必须超过一行，最后一行显然不满一行，因此没有效果。要解决最后一行列表无法两端对齐其实也很简单，就是在列表的最后创建一个高度为0的宽度100%的透明的inline-block的标签层就可以：

    ```SCSS
    li.fix{
        overflow: hidden;
        width: 100%;
        height: 0;
        padding: 0;
    }
    ```

    注意：当元素display属性是inline-block时，元素之间在换行显示或空格分隔的情况下会有间距，为了去掉这个间距，可以使用以下方法：

    1. 移除元素间的空格

    2. 无需闭合标签

    3. 使用margin负值,margin负值的大小与上下文的字体和文字大小相关，目前这里的字体大小是默认的16px，所以设置了-4px的间隙就可以了，那如果换种字体以及字体大小就又不行了，所以这个方法也不是很完美

    4. 给父元素使用`font-size: 0;`，然后在子标签里面重设font-size即可。