---
title: 为什么使用immutable.js（不可变）？
date: 2019-5-20 09:39:01
tag: redux
category: React相关
---

### immutable.js 以一种高性能的方式提供了不可变数据类型

### immutable.js 的实用功能

- 保证对象的不可变性，**所有操作都会返回一个新的对象（引用地址发生变化）** ，同时解决这种深拷贝带来的性能问题

- 提供深比较两个对象的api：Immutable.is()方法
    
    ```javascript
    //和js中对象的比较不同，在js中比较两个对象比较的是地址，
    //但是在Immutable中比较的是这个对象hashCode和valueOf，
    //只要两个对象的hashCode相等，值就是相同的，避免了深度遍历，提高了性能。
    import { Map, is } from 'immutable'
    const map1 = Map({ a: 1, b: 1, c: 1 })
    const map2 = Map({ a: 1, b: 1, c: 1 })
    map1 === map2   //false
    Object.is(map1, map2) // false
    is(map1, map2) // true
    ```
### 在 React 里面的应用（性能优化的场景）

> 目的：减少不必要的 re-render

props 变化或者 state 变化，都会导致更新流程被触发，为了减少不必要的 re-render，总体上有以下两种优化方案：

1. 使用 Component 和 shouldComponentUpdate 结合 equals 方法去进行优化

```
equals 方法会对两个对象进行深比较，引用地址不同但是内容完全相同的也被视为相同，用
```

2. 使用 PureComponent 结合 Immutable.js ，自动优化

```
PureComponent 实现了 shouldComponentUpdate 方法，通过浅比较的结果来决定是否 re-render

结合 immutable.js 之后，由于每次修改都会返回一个新的引用，因此只要 "===" 返回 false ，就说明数据发生了变化，就需要重新渲染。
```

### 在 Redux 里面的应用（浅比较新旧state，使用immutable可以保证state安全更新）

```
1. combineReducers 遍历所有的 reducer 键值对，根据每一个 reducer 返回的 state 层，构建出一个新的根 state 对象。

2. 每次循环中，combineReducers 会浅比较当前 state 层与 reducer 返回的 state 层，如果 reducer 返回了新的对象，他们就不是浅相等的，同时设置 hasChanged 为 true。

3. 循环结束后检查 hasChanged 的值，如果为 true，就会返回新构建的 state 对象，否则返回当前 state 对象
```

### 在 React-Redux 里面的应用

参考：[connect 的实现](http://huziketang.mangojuice.top/books/react/lesson38)

```javascript
export const connect = (mapStateToProps) => (WrappedComponent) => {
  class Connect extends Component {
    static contextTypes = {
      store: PropTypes.object
    }

    constructor () {
      super()
      this.state = { allProps: {} }
    }

    componentWillMount () {
      const { store } = this.context
      this._updateProps()
      store.subscribe(() => this._updateProps()) // 监听 store 的变化
    }

    _updateProps () {
      const { store } = this.context
      let stateProps = mapStateToProps(store.getState(), this.props) // 额外传入 props，让获取数据更加灵活方便
      this.setState({
        allProps: { // 整合普通的 props 和从 state 生成的 props
          ...stateProps,
          ...this.props
        }
      })
    }

    render () {
      return <WrappedComponent {...this.state.allProps} />
    }
  }

  return Connect
}
```

React-Redux 使用浅比较来决定它包装的组件是否需要重新渲染。

React-Redux 的 `connect` 方法生成的组件通过 **浅比较根 state 的引用变化** 与 `mapStateToProps` 函数的返回值，来判断包装的组件是否需要重新渲染。

connect 订阅根 state 的更新，当根 state 不变时，流程结束;

当根 state 变化时，观察 mapStateToProps 返回值是否变化，不变，则不更新 props，变的话更新props，子组件更新。

### 总结：用到浅比较的地方都需要不可变类型

React 本身使用不可变数据，则引用类型作为 props 传递时，不用担心某个修改不会触发渲染。

Redux 中使用 combineReducers，对新旧 state 进行 "===" 比较，即只比较引用地址。使用 immutable.js 同样可以保证，只要数据变化了，一定返回新的地址，也就一定会返回新的 state

React-Redux 中使用的 connect ，比较 state 用的是 "===", 比较 mapStateToProps 的返回值也是用单层"==="


> 补充：**React 中浅比较的使用场景**：

- PureComponent 的 shouldComponentUpdate

- 每个 reducer 的返回值与旧值的比较

- mapStateToProps 返回值的比较

```javascript
// 用原型链的方法
const hasOwn = Object.prototype.hasOwnProperty

// 这个函数实际上是ES6中Object.is()的polyfill，即"==="的增强版，让 +0 != -0，NaN === NaN
function is(x, y) {
  if (x === y) {
    return x !== 0 || y !== 0 || 1 / x === 1 / y
  } else {
    return x !== x && y !== y
  }
}

export default function shallowEqual(objA, objB) {
  // 首先对基本数据类型的比较
  if (is(objA, objB)) return true
  // 由于Obejct.is()可以对基本数据类型做一个精确的比较， 所以如果不等
  // 只有一种情况是误判的，那就是object,所以在判断两个对象都不是object
  // 之后，就可以返回false了
  if (typeof objA !== 'object' || objA === null ||
      typeof objB !== 'object' || objB === null) {
    return false
  }

  // 过滤掉基本数据类型之后，就是对对象的比较了
  // 首先拿出key值，对key的长度进行对比

  const keysA = Object.keys(objA)
  const keysB = Object.keys(objB)

  // 长度不等直接返回false
  if (keysA.length !== keysB.length) return false
  // key相等的情况下，在去循环比较
  for (let i = 0; i < keysA.length; i++) {
  // key值相等的时候
  // 借用原型链上真正的 hasOwnProperty 方法，判断ObjB里面是否有A的key的key值
  // 属性的顺序不影响结果也就是{name:'daisy', age:'24'} 跟{age:'24'，name:'daisy' }是一样的
  // 最后，对对象的value进行一个基本数据类型的比较，返回结果
    if (!hasOwn.call(objB, keysA[i]) ||
        !is(objA[keysA[i]], objB[keysA[i]])) {
      return false
    }
  }

  return true
}
```

> 浅比较总结（即：使用===进行比较，分整体和单层的情况）：

- **引用不同： 返回 false**

- 引用相同，第一层的key的长度不同：返回 false

- 引用相同，第一层的key的长度相同，接着使用 "===" 依次比较每个 key 对应的 value 是否相等：

    全相等：true
    
    任何不相等：false

### connect 工作原理

1. 正常初始化，render，挂载

2. 在 componentDidMount 钩子中，通过 store.subscrible 订阅 store 的变化；当store变化时，比较 mapStateToProps 返回值是否发生变化，变化的话，执行本组件的 setState 方法，重新渲染 render 中 return 的子组件。

3. **比较前后 state 变化用的是 "===" 运算符，比较 mapStateToProps 返回值用的是浅(一层)比较**

```javascript
    // connect 部分源码

    handleChange() {
        // 判断是否已经取消订阅
        if (!this.unsubscribe) {
          return
        }
    
        // 如果当前状态和上次状态相同，退出
        const storeState = this.store.getState()
        const prevStoreState = this.state.storeState
        if (pure && prevStoreState === storeState) {
          return
        }
    
        if (pure && !this.doStatePropsDependOnOwnProps) {
          // 当前状态和上次状态浅（第一层）比较：updateStatePropsIfNeeded
          const haveStatePropsChanged = tryCatch(this.updateStatePropsIfNeeded, this)
          // 如果没有变化，退出
          if (!haveStatePropsChanged) {
            return
          }
          // 比较出错
          if (haveStatePropsChanged === errorObject) {
            this.statePropsPrecalculationError = errorObject.value
          }
          // 需要预计算
          this.haveStatePropsBeenPrecalculated = true
        }
    
        // 标记store发生变化(初始化)
        this.hasStoreStateChanged = true
        // 重新改变state,也就重新触发render
        this.setState({ storeState })
    }
    
    // ！！！！ 浅比较 props （比较第一层）是否有变化 ！！！！
    updateStatePropsIfNeeded() {
        const nextStateProps = this.computeStateProps(this.store, this.props)
        if (this.stateProps && shallowEqual(nextStateProps, this.stateProps)) {
          return false
        }
        
        // 更新stateProps为下一个计算后的state
        this.stateProps = nextStateProps
        return true
    }
```

> 最后总结：

正是因为 React、Redux 中有诸多使用浅比较（直接"\=\=="或者单层比较"\=\=="）的场景存在，使用 immutable 能保证每次操作都返回一个新的对象， 每一次 state 或者 props 变化时页面都能够重新渲染。