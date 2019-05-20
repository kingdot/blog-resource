---
title: react视图更新过程
date: 2019-5-20 09:39:01
tag: react原理
category: React
---

### React 原生更新方式

首先，react是一个view（视图层）框架。能引起视图变更的只有两个地方：ReactDoM.render(jsx，real-dom)和 setState方法。

程序首次运行时，使用 `ReactDoM.render` 渲染根组件, 这个过程会递归完成所有子组件的渲染和挂载，最后根组件才挂载完毕。

在接下来的更新流程中，如果某个组件调用了 setState 方法，那么就走到该组件及其子组件的 render 流程，返回一个新的 vdom，diff 后更新real dom。

### 为什么 setState 存在两种更新方式

有可能在组件的生命周期中调用了 setState 方法，因此，要把在组件生命周期的调用缓存起来，等到事务结束时一起批量更新

### 使用 Redux 之后的更新方式：

> 状态统一管理，数据流清晰

```javascript
// 定一个 reducer
function reducer (state, action) {
  /* 初始化 state 和 switch case */
}

// 生成 store
const store = createStore(reducer)

// 监听数据变化重新渲染页面
store.subscribe(() => renderApp(store.getState())) // 重新渲染根组件，浪费性能，且子组件需要的数据也得一级一级的传下去
// 当前可以在子组件导入 store，实现精准更新，不过这样的话组件和 store 紧耦合，不好。

// 首次渲染页面
renderApp(store.getState()) 

// 后面可以随意 dispatch 了，页面自动更新
store.dispatch(...)
```

### 使用 React-Redux 之后的更新方式

> 引入 Provider 和 connect，带来以下改进：

1. 使用 context保存和传递数据

2. 容器组件订阅 store 的变化，针对性渲染，不再是每次都渲染根组件 