---
title: http常用概念总结
date: 2019-1-17 09:45:58
tag: http、https
category: http
---
`TCP/IP` 或者`SOCKET`传输的是以字节表示(即二进制)的数据包。一个字节8个bit，4bit用一个16进制数表示，刚好一个字节两位16进制表示。

HTTP请求头和响应头都是以ASCII文本方式传输的，但传输的内容可以是多样的，**服务端根据请求头中的content-type字段来获得请求中的消息主体是用何种方式编码，再对主体进行解析**。所以说到POST提交数据方案，包含了Content-Type和消息主体编码两部分。

参考：[POST提交数据详解](https://imququ.com/post/four-ways-to-post-data-in-http.html)

### 一、HTTP方法
HTTP/1.1协议规定的HTTP请求方法有**OPTIONS, GET, HEAD, POST, PUT, DELETE, TRACE, CONNECT**这几种。其中POST一般用来向服务端提交数据，本文主要讨论POST提交数据的几种方式。

#### 1.application/x-www-form-urlencode
浏览器的原生<form>表单，如果不设置`enctype`属性，那么最终就会以application/x-www-form-urlencoded方式提交数据。
```
POST http://www.example.com HTTP/1.1
Content-Type: application/x-www-form-urlencoded;charset=utf-8

title=test&sub%5B%5D=1&sub%5B%5D=2&sub%5B%5D=3
```
首先，Content-Type 被指定为 application/x-www-form-urlencoded；其次，提交的数据按照 key1=val1&key2=val2 的方式进行编码，key 和 val 都进行了 URL 转码。大部分服务端语言都对这种方式有很好的支持。

很多时候，我们用 Ajax 提交数据时，也是使用这种方式。例如 JQuery 和 QWrap 的 Ajax，Content-Type 默认值都是「application/x-www-form-urlencoded;charset=utf-8」。

#### 2. multipart/form-data
这又是一个常见的 POST 数据提交的方式。我们使用表单上传文件时，必须让 <form> 表单的 enctype 等于 multipart/form-data。直接来看一个请求示例：

```
POST http://www.example.com HTTP/1.1
Content-Type:multipart/form-data; boundary=----WebKitFormBoundaryrGKCBY7qhFd3TrwA

------WebKitFormBoundaryrGKCBY7qhFd3TrwA
Content-Disposition: form-data; name="text"

title
------WebKitFormBoundaryrGKCBY7qhFd3TrwA
Content-Disposition: form-data; name="file"; filename="chrome.png"
Content-Type: image/png

PNG ... content of chrome.png ...
------WebKitFormBoundaryrGKCBY7qhFd3TrwA--
```
这个例子稍微复杂点。首先生成了一个 boundary 用于分割不同的字段，为了避免与正文内容重复，boundary 很长很复杂。然后 Content-Type 里指明了数据是以 multipart/form-data 来编码，本次请求的 boundary 是什么内容。消息主体里按照字段个数又分为多个结构类似的部分，每部分都是以 --boundary 开始，紧接着是内容描述信息，然后是回车，最后是字段具体内容（文本或二进制）。如果传输的是文件，还要包含文件名和文件类型信息。消息主体最后以 --boundary-- 标示结束。

这种方式一般用来上传文件，各大服务端语言对它也有着良好的支持。

上面提到的这两种 POST 数据的方式，都是浏览器原生支持的，**而且现阶段标准中原生 <form> 表单也只支持这两种方式**（通过 <form> 元素的 enctype 属性指定，默认为 application/x-www-form-urlencoded。其实 enctype 还支持 text/plain，不过用得非常少）。

#### 3. application/json
application/json 这个 Content-Type作为响应头大家肯定不陌生。实际上，现在越来越多的人把它**作为请求头，用来告诉服务端消息主体是序列化后的 JSON 字符串**。由于 JSON 规范的流行，除了低版本 IE之外的各大浏览器都原生支持 JSON.stringify，服务端语言也都有处理 JSON 的函数，使用 JSON 不会遇上什么麻烦。

JSON 格式支持比键值对复杂得多的结构化数据，这一点也很有用。记得我几年前做一个项目时，需要提交的数据层次非常深，我就是把数据 JSON 序列化之后来提交的。不过当时我是**把 JSON 字符串作为 val，仍然放在键值对里，以 x-www-form-urlencoded 方式提交。**(比使用纯粹的json字符串+application/json的提交方式适用性更广)。

Google 的 AngularJS 中的 Ajax 功能，默认就是提交 JSON 字符串。例如下面这段代码：

```
var data = {'title':'test', 'sub' : [1,2,3]};
$http.post(url, data).success(function(result) {
    ...
});
```
最终发送的请求是：

```
POST http://www.example.com HTTP/1.1 
Content-Type: application/json;charset=utf-8

{"title":"test","sub":[1,2,3]}
```
这种方案，可以方便的提交复杂的结构化数据，特别适合 RESTful 的接口。各大抓包工具如 Chrome自带的开发者工具、Firebug、Fiddler，都会以树形结构展示 JSON数据，非常友好。但也**有些服务端语言还没有支持这种方式**，例如 php 就无法通过 $\_POST 对象从上面的请求中获得内容。这时候，需要自己动手处理下：在请求头中 Content-Type 为 application/json 时，从 php://input 里获得原始输入流，再 json_decode 成对象。一些 php 框架已经开始这么做了。

当然 AngularJS 也可以配置为使用 x-www-form-urlencoded 方式提交数据。

### 二、表单提交实战
#### 当前台提交的数据只是纯粹的key-value时，可以使用默认的application/x-www-urlencode编码方式
    ```javascript
    let data = {
        "key1":"value1",
        "key2":"value2",
        "key3":"value3",
    }
    let data = "key1=1&key2=2&key3=3"
    ```
#### 当前台提交的数据里面包含复杂类型，比如：数组、对象等, 有以下两种方式
##### 1. 使用application/json + 序列化的json
    ```javascript
    let data = {
        "key1":[1,2,3],
        "key2":"value2",
        "key3":{
            "key33": 4
        },
    }
    ```
     后端接收；
    ```java
    //后台设置一个结构等同于前台提交数据的model类，去接收数据
    ```
##### 2. 使用multipart/form-data
    ```javascript
    let formData = new FormData();
    // FormData 只能接收string和blob类型的数据
    // 所以要传递数组或者json对象的话，也需要先序列化成字符串
    formData.append('key1',JSON.stringify({key1:1}));
    formData.append('key2',file);
    ```
    后端接收；
    ```java
    String dataStr = request.getParameter("key1");
    JSONObject obj = JSONObject(dataStr);
    
    // 当然，也可以设置一个只含String和MultipartFile的model类去接收
    ```
##### 3. 把要发送的json**字符串**作为value，仍然使用x-www-urlencode方法
    ```javascript
    let data = {
        "key1":[1,2,3],
        "key2":"value2",
        "key3":{
            "key33": 4
        },
    };
    let dataStr = JSON.stringify(data);
    > &data=dataStr
    ```
    后端接收；
    ```java
    String dataStr = request.getParameter("data");
    JSONObject obj = JSONObject(dataStr);
    ```
