---
title: react事件绑定和传参
date: 2019-3-20 11:39:01
tag: react
category: JavaScript
---
### 一、事件绑定方式

#### ①在构造函数中绑定this,例如：

```javascript
constructor(props){
        super(props);
        this.pageGoto=this.pageGoto.bind(this);
    }
```

#### ②如果您使用实验性的 属性初始化语法，那么你可以使用属性初始值设置来正确地 绑定(bind) 回调：

```javascript
class LoggingButton extends React.Component {
  // 这个语法确保 `this` 绑定在 handleClick 中。
  // 警告：这是 *实验性的* 语法。
  handleClick = () => {
    console.log('this is:', this);
  }
  render() {
    return (
      <button onClick={this.handleClick}>
        Click me
      </button>
    );
  }
}
```

#### ③箭头函数的方式：

```javascript
<button onClick={(e) => this.handleClick(e)}>
        Click me
</button>
```

> 这个语法的问题是，每次 LoggingButton 渲染时都创建一个不同的回调。在多数情况下，没什么问题。然而，如果这个回调被作为 prop(属性) 传递给下级组件，这些组件可能需要额外的重复渲染。我们通常建议在构造函数中进行绑定，以避免这类性能问题。

### 二、传参问题

> 1. 单纯传递event对象，以下方式能良好工作：

```javascript
//  方法一：（推荐）
    // 事件对象会作为最后一个参数隐式传递
    deleteRow = (e) => {}
    <button onClick={this.deleteRow}>Delete Row</button>

// 方法二：
    // 回调函数没有绑定this时
    deleteRow(e){ }
    <button onClick={(e) => this.deleteRow(id, e)}>Delete Row</button>
    <button onClick={this.deleteRow.bind(this)}>Delete Row</button>
```



> 2. 在循环内部，通常需要将一个额外的参数传递给事件处理程序。 例如，如果 id 是一个内联 ID，则以下任一方式都可以正常工作：  
（注：不建议在循环中定义回调，应使用事件代理）

```javascript
// 不管回调函数有没有绑定this，只要传递额外参数，都不外乎以下两种方式
<button onClick={(e) => this.deleteRow(id, e)}>Delete Row</button>
<button onClick={this.deleteRow.bind(this, id)}>Delete Row</button>
```

上述两行代码是等价的，分别使用 `·arrow functions`和 `Function.prototype.bind`。

上面两个例子中，**参数 e 作为 React 事件对象将会被作为第二个参数进行传递**。

**通过箭头函数的方式，事件对象必须显式的进行传递，但是通过 bind的方式，事件对象以及更多的参数将会被隐式的进行传递。**