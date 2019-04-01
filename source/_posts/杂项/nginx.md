---
title: nginx
date: 2019-1-16 11:39:01
tag: nginx
category: 杂项
---
#### 主要用途：

##### 1. http服务器（静态资源服务器）

##### 2. 反向代理

##### 3. 负载均衡

##### 4. 隐藏后端设施


#### 反向代理的配置

在http节点下，使用upstream配置服务地址，使用server的location配置代理映射。

```JavaScript
upstream my_server {        // 配置服务器地址                                                 
    server 10.0.0.2:8080;                                                
    keepalive 2000;
}
server {
    listen       80;                                                         
    server_name  10.0.0.1;  // 客户端请求的地址                                             
    client_max_body_size 1024M;

    location /my/ {
        proxy_pass http://my_server/;  // 转向目标服务器
        proxy_set_header Host $host:$server_port; // 给响应添加nginx本身的host和port，起到隐藏后部服务设施的目的
    }
}
```

通过该配置，访问nginx地址http://10.0.0.1:80/my的请求会被转发到my_server服务地址http://10.0.0.2:8080/。

> 需要注意的是，如果按照如下配置：

```
upstream my_server {                                                         
    server 10.0.0.2:8080;                                                
    keepalive 2000;
}
server {
    listen       80;                                                         
    server_name  10.0.0.1;                                               
    client_max_body_size 1024M;

    location /my/ {
        proxy_pass http://my_server;
        proxy_set_header Host $host:$server_port;
    }
}
```

那么，访问nginx地址http://10.0.0.1:80/my的请求会被转发到my_server服务地址http://10.0.0.2:8080/my。这是因为proxy_pass参数中如果不包含url的路径，则会将location的pattern识别的路径作为绝对路径。

