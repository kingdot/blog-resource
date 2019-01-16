---
title: cookie相关
date: 2019-1-16 11:39:01
tag: JavaScript核心
category: JavaScript
---
#### 介绍：
Cookie是由HTTP服务器设置的，保存在浏览器中的文件，用于弥补http协议无状态的不足。

#### 分类：
- 会话cookie：不设置它的生命周期expires时的状态，浏览器的开关就是一次会话。
- 持久cookie：在过期前都有效，且cookie会自动将数据传送到服务器端。

#### 格式
cookie是一个以等号分割key和value，不同cookie用分号加空格（; ）分开的字符串。
- 常见用法：
    > cookie的几种常见属性：document.cookie="key=value;expires=失效时间;path=路径;domain=域名;secure;(secure表安全级别）
- 比如：
    > YNOTE_LOGIN=3||1533575276767; YNOTE_CSTK=UMerv24j;

#### 使用：
- 读取所有可从此位置访问的Cookie：allcookie = document.cookie
- 设置一个（新）cookie：document.cookie = ***，此方法一次只能对一个cookie进行设置或更新

实例：

```
// key : cookie 名
// value : cookie 值
// options : 可选配置参数
//		options = {
//			expires : 7|new Date(), // 失效时间
//			path : "/", // 路径
//			domain : "", // 域名
//			secure : true // 安全连接
//		}
function cookie(key, value, options) {
	/* read 读取 */
	// 如果没有传递 value ，则表示根据 key 读取 cookie 值
	if (typeof value === "undefined") { // 读取
		// 获取当前域下所有的 cookie，保存到 cookies 数组中
		var cookies = document.cookie.split("; ");
		// 遍历 cookies 数组中的每个元素
		for (var i = 0, len = cookies.length; i < len; i++) {
			// cookies[i] : 当前遍历到的元素，代表的是 "key=value" 意思的字符串，
			// 将字符串以 = 号分割返回的数组中第一个元素表示 key，
			// 第二个元素表示 value
			var cookie = cookies[i].split("=");
			// 判断是否是要查找的 key，对查找的 key 、value 都要做解码操作
			if (decodeURIComponent(cookie[0]) === key) {
				return decodeURIComponent(cookie[1]);
			}
		}
		// 没有查找到指定的 key 对应的 value 值，则返回 null
		return null;
	}
 
	/* 存入 设置 */
	// 设置 options 默认为空对象
	options = options || {};
	// key = value，对象 key，value 编码
	var cookie = encodeURIComponent(key) + "=" + encodeURIComponent(value);
	// 失效时间
	if ((typeof options.expires) !== "undefined") { // 有配置失效时间
		if (typeof options.expires === "number") { // 失效时间为数字
			var days = options.expires, 
				t = options.expires = new Date();
			t.setDate(t.getDate() + days);
		} 
		cookie += ";expires=" + options.expires.toUTCString();
	}
	// 路径
	if (typeof options.path !== "undefined")
		cookie += ";path=" + options.path;
	// 域名
	if (typeof options.domain !== "undefined")
		cookie += ";domain=" + options.domain;
	// 安全连接
	if (options.secure)
		cookie += ";secure";
 
	// 保存
	document.cookie = cookie;
}
 
// 从所有的 cookie 中删除指定的 cookie
function removeCookie(key, options) {
	options = options || {};
	options.expires = -1; // 将失效时间设置为 1 天前
	cookie(key, "", options);
}
```