---
title: 从输入url到页面展现发生了什么
date: 2019-5-14 21:39:01
tag: JavaScript基础
category: JavaScript
---

1. 在浏览器地址栏输入URL

2. 浏览器查看缓存，如果请求资源在缓存中并且新鲜，跳转到转码步骤

    1. 如果资源未缓存，发起新请求

    2. 如果已缓存，检验是否足够新鲜，足够新鲜直接提供给客户端，否则与服务器进行验证。

        检验新鲜通常有两个HTTP头进行控制Expires和Cache-Control：
        
            HTTP1.0提供Expires，值为一个绝对时间表示缓存新鲜日期
            
            HTTP1.1增加了Cache-Control: max-age=,值为以秒为单位的最大新鲜时间

3. 浏览器解析URL获取协议，主机，端口，path

4. 浏览器组装一个HTTP（GET）请求报文

5. 浏览器获取主机ip地址，过程如下：

    1. 浏览器缓存
    
    2. 本机缓存
    
    3. hosts文件
    
    4. 路由器缓存
    
    5. ISP DNS缓存
    
    6. DNS递归查询（可能存在负载均衡导致每次IP不一样）
    
6. 打开一个socket与目标IP地址，端口建立TCP链接，三次握手如下（前两次有syn）：
    
    ```
    客户端发送一个TCP的SYN=1，Seq=X的包到服务器端口
    
    服务器发回SYN=1， ACK=X+1， Seq=Y的响应包
    
    客户端发送ACK=Y+1， Seq=Z
    
    （补充：3次握手的目的主要是防止失效的握手报文抵达，使server陷入等待）
    ```    
    
7. TCP链接建立后发送HTTP请求

8. 服务器接受请求并解析，将请求转发到服务程序，如虚拟主机使用HTTP Host头部判断请求的服务程序（反向代理的情况）

9. 服务器检查HTTP请求头是否包含缓存验证信息如果验证缓存新鲜，返回304等对应状态码

10. 处理程序读取完整请求并准备HTTP响应，可能需要查询数据库等操作

11. 服务器将响应报文通过TCP连接发送回浏览器

12. 浏览器接收HTTP响应，然后根据情况选择关闭TCP连接或者保留重用，关闭TCP连接的四次握手如下：
    
    ```
    主动方发送Fin=1， Ack=Z， Seq= X报文
    
    被动方发送ACK=X+1， Seq=Z报文
    
    被动方发送Fin=1， ACK=X， Seq=Y报文
    
    主动方发送ACK=Y， Seq=X报文
    
    (补充：前两次是主动方告诉被动方，主动方不再发送数据了，此时被动方还可以给主动方发数据；后两次说明被动方也没有要发送的数据了)
    ```
    
13. 浏览器检查响应状态吗：是否为1XX，3XX， 4XX， 5XX，这些情况处理与2XX不同

    1. 如果资源可缓存，进行缓存

    2. 对响应进行解码（例如gzip压缩）

14. 根据资源类型决定如何处理（假设资源为HTML文档）

    解析HTML文档，构件DOM树，下载资源，构造CSSOM树，执行js脚本，这些操作没有严格的先后顺序，以下分别解释
    
    参考：[渲染树构建、布局及绘制](https://developers.google.com/web/fundamentals/performance/critical-rendering-path/render-tree-construction?hl=zh-cn)
    
    - 构建DOM树：
    
        Tokenizing：根据HTML规范将字符流解析为标记
        
        Lexing：词法分析将标记转换为对象并定义属性和规则
        
        DOM construction：根据HTML标记关系将对象组成DOM树
        
        解析过程中遇到图片、样式表、js文件，启动下载

    - 构建CSSOM树：
    
        Tokenizing：字符流转换为标记流
        
        Node：根据标记创建节点
        
        CSSOM：节点创建CSSOM树
        
        （补充：css媒体类型和媒体查询不会阻塞渲染，但都会下载）

    - 根据DOM树和CSSOM树构建渲染树:
    
        从DOM树的根节点遍历所有可见节点，不可见节点包括：
        
            1）script,meta这样本身不可见的标签。
            
            2)被css隐藏的节点，如display: none
        
        对每一个可见节点，找到恰当的CSSOM规则并应用
        
    - 发布可视**节点的内容和计算样式**

    - 进入layout布局阶段
    
        到目前为止，我们计算了哪些节点应该是可见的以及它们的计算样式，但我们**尚未计算它们在设备视口内的确切位置和大小**---这就是“布局”阶段，也称为“自动重排”。

        为弄清每个对象在网页上的确切大小和位置，浏览器从渲染树的根节点开始进行遍历。
        
        **布局流程的输出是一个“盒模型”，它会精确地捕获每个元素在视口内的确切位置和尺寸**：所有相对测量值都转换为屏幕上的绝对像素。
    
    - 进入painting绘制阶段
    
        最后，既然我们知道了哪些节点可见、它们的计算样式以及几何信息，我们终于可以将这些信息传递给最后一个阶段：将渲染树中的每个节点转换成屏幕上的实际像素。这一步通常称为“绘制”或“栅格化”。
    
> 补充： webkit主流程
    
![](https://segmentfault.com/img/bVRm39?w=624&h=289)
    
1. 解析HTML，生成DOM树，解析CSS，生成CSSOM树

2. 将DOM树和CSSOM树结合，生成渲染树(Render Tree)

3. Layout:根据生成的渲染树，进行Layout，得到节点的几何信息（位置，大小）

4. Painting:根据渲染树以及回流得到的几何信息，得到节点的绝对像素

5. Display:将像素发送给GPU，展示在页面上。（这一步其实还有很多内容，比如会在GPU将多个合成层合并为同一个层，并展示在页面中。而css3硬件加速的原理则是新建合成层，这里我们不展开
    
> 补充：一些易混淆的点

1. 关于阻塞：

由上图可知，cssom树和dom树是并行构建的，即css不阻塞DOM；cssdom和dom都位于render前面，因此二者都阻塞页面渲染

js的下载和执行会阻塞dom的构建，但可以通过defer和async让下载过程不阻塞，但执行过程一定阻塞（因为js可以修改dom）。除了async的脚本不会阻塞DOMContentLoaded,其余类型脚本都会阻塞。

css样式的下载不会影响js的下载，但会阻塞js的执行(因为js可能会获取样式)；css不会阻塞dom树的构建，因此继续向下解析，有可能解析到script时css还没处理完，则此js的执行会被阻塞。

**总结：CSS不阻塞DOM构建而阻塞JS脚本执行，JS脚本下载（可解决）和执行阻塞DOM树构建**
        
2. DOM树和渲染树的区别

渲染树实际上就是一个计算好样式，与HTML对应的（包括哪些显示，那些不显示）的Tree。它是由渲染对象组成的，每个渲染对象都有一个布局（reflow）方法，实现其布局或回流。

3. 白屏时间和首屏时间

白屏时间指head标签解析完毕，开始解析body之前的时间

首屏时间指首屏的所有资源都加载完毕的时间

4. 我们常常提到的reflow和repaint是什么？

从上面的流程中，可以看到页面初始化时，至少有一次layout和paint；这个过程是不可缺少的，那么我们通常所提到的reflow和repaint是什么？

对渲染树的布局可以分为全局和局部的，全局即对整个渲染树进行重新布局，如当我们改变了窗口尺寸或方向或者是修改了根元素的尺寸或者字体大小等；而局部布局可以是对渲染树的某部分或某一个渲染对象进行重新布局。

一旦cssom或者dom被修改，即渲染树被修改，该变化的渲染对象需要layout；当然，这次layout可能会导致相邻或者父子关系的其他节点的reflow，当渲染树变化较多或者某些操作触发时，会触发整个页面的layout。

> 补充：html解析过程触发的事件

1. 浏览器创建Document对象并解析HTML，将解析到的元素和文本节点添加到文档中，此时document.readystate为loading

1. HTML解析器遇到没有async和defer的script时，将他们添加到文档中，然后执行行内或外部脚本。这些脚本会同步执行，并且在脚本下载和执行时解析器会暂停。这样就可以用document.write()把文本插入到输入流中。同步脚本经常简单定义函数和注册事件处理程序，他们可以遍历和操作script和他们之前的文档内容

1. 当解析器遇到设置了async属性的script时，开始下载脚本并继续解析文档。脚本会在它下载完成后尽快执行，但是解析器不会停下来等它下载。异步脚本禁止使用document.write()，它们可以访问自己script和之前的文档元素

1. 当文档完成解析，document.readState变成interactive
所有defer脚本会按照在文档出现的顺序执行，延迟脚本能访问完整文档树，禁止使用document.write()

1. 浏览器在Document对象上触发DOMContentLoaded事件

1. 此时文档完全解析完成，浏览器可能还在等待如图片等内容加载，等这些内容完成载入并且所有异步脚本完成载入和执行，document.readState变为complete,window触发load事件

1. 显示页面（HTML解析过程中会逐步显示页面）

> 补充：影响layout和repaint的属性

影响 layout 的属性

| 宽高 | 边距 | 位置 | 表现 | 边框 | 定位 | 字体 |
|:-:|:-:|:-:|:-:|:-:|:-:|:-:|
|width| padding| position| display| border| text-align| font-size|
|height| margin| top|float| border-width| overflow-y| font-weight|

影响repaint的属性

|背景| 边框 | 其他 |
|:-:|:-:|:-:|
|background| border-style| color|
|background-image| border-radius| visibility|
|background-repeat| outline| text-decoration|
|background-position| outline-style| box-shadow|
|background-size| outline-color|
– |outline-width


> 补充：css3动画优劣

1. 好处：

    - 使用css3硬件加速，可以让transform、opacity、filters这些动画不会引起回流重绘
    
    - 对于动画的其它属性，比如background-color这些，还是会引起回流重绘的，不过它还是可以提升这些动画的性能。

2. css3硬件加速的坑:

    - 如果你为太多元素使用css3硬件加速，会导致内存占用较大，会有性能问题。
    
    - 在GPU渲染字体会导致抗锯齿无效（导致动画里的字体是模糊的）。这是因为GPU和CPU的算法不同。因此如果你不在动画结束的时候关闭硬件加速，会产生字体模糊。


https://segmentfault.com/a/1190000017329980

https://segmentfault.com/a/1190000010298038#articleHeader4