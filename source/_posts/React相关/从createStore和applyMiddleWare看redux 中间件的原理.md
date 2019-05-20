---
title: 从createStore和applyMiddleWare看redux 中间件的原理
date: 2019-5-20 09:39:01
tag: redux
category: React相关
---


### createStore 部分源码

```javascript
// createStore
// enhancer 为 applyMiddleWare 的执行结果，即 applyMiddleWare([middleware1, middleware2])
export default function createStore(reducer, preloadedState, enhancer) {

  // 处理只有两个参数的情况    
  if (typeof preloadedState === 'function' && typeof enhancer === 'undefined') {
    enhancer = preloadedState
    preloadedState = undefined
  }

  // enhancer 为 function，创建 store 的流程转交给 applyMiddleWare 控制
  if (typeof enhancer !== 'undefined') {
    if (typeof enhancer !== 'function') {
      throw new Error('Expected the enhancer to be a function.')
    }
    // 给 applyMiddleWare 传入 createStore 后执行applyMiddleWare
    // 给 执行后返回的函数传入 reducer 和 state
    return enhancer(createStore)(reducer, preloadedState)
  }
  
  // 没有传入 enhancer 的情况，即无需修改 dispatch，此时我们来定义 store 相关的内容，包括
  // store.getState()
  // store.dispatch()
  // store.subscribe()
  // store.replaceReducer()
  
  // ... ...
}

// applyMiddleWare，目的：返回一个 enhancer
// enhancer 的目的：创建一个 dispatch 被替换的新的store
export default function applyMiddleware(...middlewares) {
  // 返回的函数即为 enhancer
  return createStore => (...args) => {
    // 调用无 enhancer 参数的 createStore 函数，生成标准 store
    const store = createStore(...args)
    // 改造 dispatch 完成前，不允许调用 dispatch
    let dispatch = () => {
      throw new Error(
        'Dispatching while constructing your middleware is not allowed. ' +
          'Other middleware would not be applied to this dispatch.'
      )
    }
    // 中间件所需的两个方法
    const middlewareAPI = {
      getState: store.getState,
      dispatch: (...args) => dispatch(...args)
    }
    // chain 里面存放的是 (next) => (action) => {...}，即接收 dispatch ，并改造之的函数
    // 以 redux-thunk 为例，此处传给中间件 {getState, dispatch} 后执行，返回一个接收 next 的函数
    const chain = middlewares.map(middleware => middleware(middlewareAPI))
    // 即：dispatch = a(b(c(d(dispatch)))), 
    // 中间件 d 用的是原生 dispatch，
    // 中间件 c 用的是 d 返回的 dispatch
    // ... ...
    // 注意：每一个的执行结果都是一个 dispatch
    // 即：a(b(c(d(dispatch)))) 可以看作：
    // a(dispatch_b) b(dispatch_c) c(dispatch_d) d(dispatch)
    // 视图层在执行 dispatch 的时候，实际执行的是 a 改造后的 dispatch，
    // 由于 a 产生的 dispatch 需要 b 返回的 dispatch_b 当参数（即依赖 dispatch_b）, 因此需要执行 b(dispatch_c), 如此下去，直到 d 执行，再一层层返回，类似递归的过程
    dispatch = compose(...chain)(store.dispatch)

    // 返回被替换掉 dispatch 的 store
    return {
      ...store,
      // 用新的 dispatch 替换 store 里面的原生 dispatch
      dispatch
    }
  }
}

// compose
export default function compose(...funcs) {
  if (funcs.length === 0) {
   // 透传
    return arg => arg
  }

  if (funcs.length === 1) {
    return funcs[0]
  }
  // 作用：compose(a,b,c,d)(dispatch) 转换成 a(b(c(d(dispatch))))
  // 而每一个 middleware 都返回一个新的 dispatch
  // 可以看到在 dispatch 构造阶段， args 被传递给了最里层的中间件函数
  return funcs.reduce((a, b) => (...args) => a(b(...args)))
}

// redux-thunk 中间件
// 接收 dispatch 和 getState 函数
function createThunkMiddleware(extraArgument) {
  return ({ dispatch, getState }) => next => action => {
    if (typeof action === 'function') {
    // 如果 action 是一个函数，则执行它
      return action(dispatch, getState, extraArgument);
    }
    // 否则，用传入的 next 去处理该 action
    return next(action);
  };
}

const thunk = createThunkMiddleware();
thunk.withExtraArgument = createThunkMiddleware;

export default thunk;
```

### 中间件执行过程分析 

参考：[理解 redux 中间件](https://zhuanlan.zhihu.com/p/21391101)

假如有以下两个中间件：

```javascript
const logger = store => next => action => {
  console.log('dispatching', action)
  return next(action)
}

const collectError = store => next => action => {
  try {
    return next(action)
  } catch (err) {
    console.error('Error!', err)
  }
}
// 调用的时候从后往前执行
applyMiddleWare([collectError, logger])
```

> 构造过程：

```javascript
let next = store.dispatch;
let dispatch1 = logger(store)(next); 
// 这时候的console.log('dispatching', action) 是没有执行的 
let dispatch2 = collectError(store)(dispatch1);
// 这时候的console.log('Error!', err) 也是没有执行的 
store.dispatch = dispatch2;
```

> 执行过程：

dispatch 的时候，先执行 dispatch2 ，然后 dispatch2 里面使用了 next函数，即 dispatch1，所以转去执行1，1执行完成后，2继续执行，有点像递归。

### 注意事项

如果在 调用 next(action) 之前， 使用 await 调用， 会阻塞住整个中间件的链式调用，然后在 await 位置立刻返回， 造成不符合预期的结果。

如果要在中间件中使用异步操作， 需要放在 next(action) 之后， 这样不会影响其他的中间件调用， 用可以把异步的逻辑放到最后执行。