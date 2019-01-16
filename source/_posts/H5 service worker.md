---
title: H5 service worker
date: 2019-1-16 11:39:01
tag: JavaScript核心
category: JavaScript
---
## service worker

### 出现的原因：
native应用比web应用更受青睐的主要原因就是，在没有通过网络接收到更多的数据前，仍可以提供基本的功能。

### 之前的解决方案AppCache
`AppCache`通过在html页面的HTML标签上增加manifest属性，这个属性值与`缓存清单`文件关联。

应该在每个意图缓存的页面上添加manifest特性。同时，你也不需要在清单文件中列出所有你想缓存的页面。浏览器会自动的将带有manifest属性的页面缓存。

具体使用细节请参考[AppCache](https://developer.mozilla.org/en-US/docs/Web/HTML/Using_the_application_cache)

### service worker
通过navigator.serviceWorker注册一个serviceworker的url，然后就可以在serviceworker的文件中通过监听一些事件做一些行为。

使用步骤：

通常遵循以下基本步骤来使用 service workers：
1. service worker URL 通过serviceWorkerContainer.register() 来获取和注册。

2. 如果注册成功，service worker 就在 ServiceWorkerGlobalScope 环境中运行； 这是一个特殊类型的 woker 上下文运行环境，与主运行线程（执行脚本）相独立，同时也没有访问 DOM 的能力。
3. service worker 现在可以处理事件了。
4. 受 service worker 控制的页面打开后会尝试去安装 service worker。最先发送给 service worker 的事件是安装事件(在这个事件里可以开始进行填充 IndexDB和缓存站点资源)。这个流程同原生 APP 或者 Firefox OS APP 是一样的 — 让所有资源可离线访问。
5. 当 oninstall 事件的处理程序执行完毕后，可以认为 service worker 安装完成了。
6. 下一步是激活。当 service worker 安装完成后，会接收到一个激活事件(activate event)。 onactivate 主要用途是清理先前版本的service worker 脚本中使用的资源。
7. Service Worker 现在可以控制页面了，但仅是在 register()  成功后的打开的页面。也就是说，页面起始于有没有 service worker ，且在页面的接下来生命周期内维持这个状态。所以，页面不得不重新加载以让 service worker 获得完全的控制。

