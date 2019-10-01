---
title: react新特性
date: 2019-10-1 11:39:01
tag: react
category: JavaScript
---

## Hooks

### 1. State Hook 

即在函数组件中创建 `state` 和与之搭配的 `setState()` 方法

##### 用法：

> let [someProps, setSomeProps] = useState(initData)

当跟视图相关的数据发生了变化时，可以有如下两种方式触发 UI 重新渲染：

1. 把新值作为参数传给 setSomeProps 方法。

2. 传递一个函数给 setSomeProps 方法，该函数接收一个 oldVlaue 入参。

### 2. Effect Hook

在 `React` 组件中执行**数据获取**、**订阅**或者**手动修改**过 DOM 这些操作称为“副作用”。`Effect Hook` 可以让你在函数组件中执行副作用操作。

`useEffect` 就是一个 `Effect Hook`，给函数组件增加了操作副作用的能力。它跟 `class 组件` 中的 `componentDidMount`、`componentDidUpdate` 和 `componentWillUnmount` 具有相同的用途，只不过被合并成了一个 API。

**effects会在每次渲染后运行**，并且概念上它是组件输出的一部分，可以“看到”属于某次特定渲染的 `props` 和 `state`。

实际上每一个组件内的函数（包括事件处理函数，`effects`，定时器或者 `API` 调用等等）会通过闭包来捕获某次渲染中定义的 `props` 和 `state`。在组件内什么时候去读取 `props` 或者 `state` 是无关紧要的。因为它们不会改变，在单次渲染的范围内，`props` 和 `state` 始终保持不变。（解构赋值的props使得这一点更明显。）

##### 如何避免Capture Value，使得取到的数据为最新？

场景：有时候你可能想在effect的回调函数里读取最新的值而不是捕获的值。最简单的实现方法是使用 `refs`:

利用 useRef 就可以绕过 Capture Value 的特性。可以认为 ref 在所有 Render 过程中保持着唯一引用，因此所有对 ref 的赋值或取值，拿到的都只有一个最终状态，而不会在每个 Render 间存在隔离。

```javascript
function Example() {
  const [count, setCount] = useState(0);
  const latestCount = useRef(count);

  useEffect(() => {
    // Set the mutable latest value
    latestCount.current = count;
    setTimeout(() => {
      // Read the mutable latest value
      console.log(`You clicked ${latestCount.current} times`);
    }, 3000);
  });
  // ...
}
```

也可以简洁的认为，ref 是 Mutable 的，而 state 是 Immutable 的。

##### useEffect用法1：在每次UI更新后，调用effect操作副作用

> useEffect(cb)

```javascript
//cb
() => {
    //some operations
    // ...
}
```

##### useEffect用法2：做一些副作用的清除操作

```javascript
// 如果需要在下一次渲染（包括组件被销毁）时，想让通过 useEffect 注册的副作用被销毁，这一点可以通过 useEffect 的返回值做到：
useEffect(() => {
  ChatAPI.subscribeToFriendStatus(props.id, handleStatusChange);
  return () => {
    ChatAPI.unsubscribeFromFriendStatus(props.id, handleStatusChange);
  };
});
```

**一般来说：**

在下一次绘制完成之后、 effect 执行之前，执行清理工作，并且被清理的内容属于上次渲染。

假设第一次渲染的时候props是{id: 10}，第二次渲染的时候是{id: 20}，则有：

- React 渲染{id: 20}的UI。

- 浏览器绘制。我们在屏幕上看到{id: 20}的UI。

- React 清除{id: 10}的effect。

- React 运行{id: 20}的effect。

##### useEffect用法3：在不需要的时候避免调用effect

> useEffect(cb, [deps])

React并不能猜测到函数做了什么如果不先调用的话，因此 React 无法区分 effects 的不同。如果想要避免effects不必要的重复调用，你可以提供给useEffect一个依赖数组参数(deps)，通知 React 仅仅当在 deps里面的数据发生变化时调用 effects。

注意：当 deps 为一个空数组的时候，effects 仅仅在首次挂载的时候得到执行

**两种告知依赖的策略：（可结合使用）**

- 第一种策略是在依赖中包含所有effect中用到的组件内的值。
    
    ```javascript
    // 例如：
    useEffect(()=>{
        setSomePorps(count + 1)
    }, [count])
    ```

- 第二种策略是修改effect内部的代码以确保它包含的值只会在需要的时候发生变更。一般配合 `setSomeProps((oldValue)=>{...})` 来使用。

    ```javascript
    // 例如：
    useEffect(()=>{
        setInterval(()=>{
            setSomePorps(oldValue => oldVlaue + 1)
        },3000)
    }, [])
    ```

参考：

[useEffect 完整指南](https://overreacted.io/zh-hans/a-complete-guide-to-useeffect/)

[精读《useEffect 完全指南》](https://juejin.im/post/5c9827745188250ff85afe50#heading-6)

### 3. ContextHooks

Context API是React提供的一种跨节点数据访问的方式。context 很早就有，只是不好用，源自 redux 的灵感，react 从 v16.3 之后推出了全新的版本。

用法：

> useContext() 

```javascript
const CountContext = React.createContext();

const CountProvider = ({ children }) => {
  const contextValue = useReducer(reducer, initialState);
  return (
    <CountContext.Provider value={contextValue}>
      {children}
    </CountContext.Provider>
  );
};

// some v-dom
<>
    <CountProvider>
      <Counter />
    </CountProvider>
    <CountProvider>
      <Counter />
    </CountProvider>
  </>
  
// counter
const useCount = () => {
  const contextValue = useContext(CountContext);
  return contextValue;
};

const Counter = () => {
  const [count, dispatch] = useCount();
  return (
    <div>
      {count}
      <button onClick={() => dispatch({ type: "increment" })}>+1</button>
      <button onClick={() => dispatch({ type: "decrement" })}>-1</button>
      <button onClick={() => dispatch({ type: "set", count: 0 })}>reset</button>
    </div>
  );
};

```

### 4. useReducer

##### 用法一（常规用法，单纯操作state）：

> const [state, dispatch] = useReducer(reducer, initialState);

reducer可以让你把组件内发生了什么(actions)和状态如何响应并更新分开表述。即可用于解耦用户行为和状态管理。

effect不再关心怎么更新状态，它只负责告诉我们发生了什么。更新的逻辑全都交由reducer去统一处理:

```javascript
// someReducer.js
const initialState = {
  count: 0,
  step: 1,
};

// 一般情况下，reducer只能操作state
function reducer(state, action) {
  const { count, step } = state;
  if (action.type === 'tick') {
    return { count: count + step, step };
  } else if (action.type === 'step') {
    return { count, step: action.step };
  } else {
    throw new Error();
  }
}
```

React会保证dispatch在组件的声明周期内保持不变。因此，可以使用 `useReducer` 来精简依赖。

```javascript
const [state, dispatch] = useReducer(reducer, initialState);
const { count, step } = state;

useEffect(() => {
  const id = setInterval(() => {
    dispatch({ type: 'tick' }); // Instead of setCount(c => c + step);
  }, 1000);
  return () => clearInterval(id);
}, []); //写成 [dispatch] 也 ok
```

注：可以从依赖中去除 `dispatch`, `setState`, 和 `useRef` 包裹的值因为 `React` 会确保它们是静态的。不过设置了它们作为依赖也没什么问题。

##### 用法二（特殊用法，操作 state 和 props）：

```javascript
function Counter({ step }) {
  const [count, dispatch] = useReducer(reducer, 0);

  function reducer(state, action) {
    if (action.type === 'tick') {
      return state + step;
    } else {
      throw new Error();
    }
  }

  useEffect(() => {
    const id = setInterval(() => {
      dispatch({ type: 'tick' });
    }, 1000);
    return () => clearInterval(id);
  }, [dispatch]);

  return <h1>{count}</h1>;
}
```

如上所示代码，当把 reducers 放到组件内部时，就可以访问到 props 了。

#### tips：

1. 可以把跟 effect 相关的函数都写在 effect 内部，这样便于精准的书写依赖

2. 可以把函数作为依赖，此时该函数是可变的，它的描述使用 `useCallback(func, [deps])`

```javascript
//对于通过属性从父组件传入的函数适用：
function Parent() {
  const [query, setQuery] = useState('react');

  // ✅ Preserves identity until query changes
  const fetchData = useCallback(() => {
    const url = 'https://hn.algolia.com/api/v1/search?query=' + query;
    // ... Fetch data and return it ...
  }, [query]);  // ✅ Callback deps are OK

  return <Child fetchData={fetchData} />
}

function Child({ fetchData }) {
  let [data, setData] = useState(null);

  useEffect(() => {
    fetchData().then(setData);
  }, [fetchData]); // ✅ Effect deps are OK

  // ...
}
```

### 5. 自定义hooks

自定义 Hook 是一个函数，其名称以 “use” 开头，函数内部可以调用其他的 Hook。

与 React 组件不同的是，自定义 Hook 不需要具有特殊的标识。我们可以自由的决定它的参数是什么，以及它应该返回什么（如果需要的话）。换句话说，它就像一个正常的函数。但是它的名字应该始终以 use 开头，这样可以一眼看出其符合 Hook 的规则。

自定义 Hook 是一种自然遵循 Hook 设计的约定，而并不是 React 的特性。

自定义 Hook 必须以 “use” 开头。这个约定非常重要。不遵循的话，由于无法判断某个函数是否包含对其内部 Hook 的调用，React 将无法自动检查你的 Hook 是否违反了 Hook 的规则。

### 6. fragments

```javascript
// 最初始版本
<div>
  <ChildA />
  <ChildB />
  <ChildC />
</div>

// v16.2 引入fragments
const Fragment = React.Fragment;

<Fragment>
  <ChildA />
  <ChildB />
  <ChildC />
</Fragment>

// This also works
<React.Fragment>
  <ChildA />
  <ChildB />
  <ChildC />
</React.Fragment>

// 最后也可以简化成
<>
  <ChildA />
  <ChildB />
  <ChildC />
</>

// key 是唯一可以传递给 fragments 的属性
```
