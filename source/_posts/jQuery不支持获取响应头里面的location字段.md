---
title: jQuery不支持获取响应头里面的location字段
date: 2019-1-17 09:45:58
tag: http、https
category: http
---
#### jQuery不支持获取响应头里面的location字段

##### 使用场景：
前天使用ajax访问一个没有权限的接口时，服务器返回302，并且用location重定向到指定页面。
于是，在回调中这么写了：
``` javascript
success: function(data, status, xhr){
    window.location.href = xhr.getResponseHeader('Location');
}
```
然而，并没有预期效果，调试发现，`xhr.getResponseHeader('Location')`的输出是null。查资料得知，jQuery给出的xhr并不能获取到location字段。

##### 解决方案：
1. 另行设置一个自定义响应头，在xhr里面获取。
2. 使用原生XmlHttpRequest，通过xhr.responseUrl获取。