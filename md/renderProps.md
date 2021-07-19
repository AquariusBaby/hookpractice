# render Props
---

 - 严格来讲，Mixin、Render Props、HOC 等方案都只能算是在既有（组件机制的）游戏规则下探索出来的上层模式；

 - 术语 “render prop” 是指一种在 React 组件之间使用一个值为函数的 prop 共享代码的简单技术;

 - 具有 render prop 的组件接受一个函数，该函数返回一个 React 元素并调用它而不是实现自己的渲染逻辑；

```javascript
<DataProvider render={data => (
  <h1>Hello {data.target}</h1>
)}/>
```

---

### 使用 Props 而非 render

render prop 是因为模式才被称为 render prop ，你不一定要用名为 render 的 prop 来使用这种模式。事实上， 任何被用于告知组件需要渲染什么内容的函数 prop 在技术上都可以被称为 “render prop”。

很多时候，我们通常都用的是 children prop，写法上更优雅。

```javascript
<Mouse>
  {mouse => (
    <p>鼠标的位置是 {mouse.x}，{mouse.y}</p>
  )}
</Mouse>
```
---

### 注意事项

将render props 和 React.PurComponent 一起使用时，会使 React.PurComponent 带来的优化失效

问题：如果你在 render 方法里创建函数，那么使用 render prop 会抵消使用 React.PureComponent 带来的优势。因为浅比较 props 的时候总会得到 false，并且**在这种情况下每一个 render 对于 render prop 将会生成一个新的值**。

解决方式：有时你可以把这个方法作为添加为实例方法，如下：

```javascript
class MouseTracker extends React.Component {
  // 定义为实例方法，`this.renderTheCat`始终
  // 当我们在渲染中使用它时，它指的是相同的函数
  renderTheCat(mouse) {
    return <Cat mouse={mouse} />;
  }

  render() {
    return (
      <div>
        <h1>Move the mouse around!</h1>
        <Mouse render={this.renderTheCat} />
      </div>
    );
  }
}
```

---

### render props 与 HOC

类比 HOC,技术上，二者都基于组件组合机制，Render Props 拥有与 HOC 一样的扩展能力,之所以称之为 Render Props，并不是说只能用来复用渲染逻辑,而是表示在这种模式下，组件是通过render()组合起来的，类似于 HOC 模式下通过 Wrapper 的render()建立组合关系,形式上，二者非常相像，同样都会产生一层“Wrapper”（EComponent和RP）；

<!-- Render Props 与 HOC 能够相互转换： -->