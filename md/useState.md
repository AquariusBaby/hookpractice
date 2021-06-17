#useState

----

const [state, update] = useState(initVal);

  - 接受一个值或者函数，来初始化state；返回一个值及其对应的更新函数；
  - 惰性初始 state：如果初始 state 需要通过复杂计算获得，则可以传入一个函数，在函数中计算并返回初始的 state。（用于初始化的值和函数仅在初始渲染时会被调用）
  - 跳过 state 更新： 调用update更新函数并传入当前的 state 时，React 将跳过子组件的渲染及 effect 的执行。（React 使用 Object.is 比较算法 来比较 state，相同时会被跳过）；
  - update具有稳定性：React 会确保 update 函数的标识是稳定的，并且不会在组件重新渲染时发生变化，你可以从 useEffect 或 useCallback 的依赖列表中省略 update
  - update接受一个更新值或者一个函数，函数的入参为上一次的旧值；

---

**问题一：我该使用单个 state 变量还是多个 state 变量？**
useState 的出现，让我们可以使用多个 state 变量来保存 state，比如这样：
```
const [width, setWidth] = useState(100);
const [height, setHeight] = useState(100);
const [left, setLeft] = useState(0);
const [top, setTop] = useState(0);
```

也可以这样，将所有的 state 放到一个 object 中
```
const [state, setState] = useState({
  width: 100,
  height: 100,
  left: 0,
  top: 0
});
```

我们再看下class组件，是这样的：
```
class A extends React.Component {
    constructor(props) {
        super(props);
        this.timer = null; // 和视图无关的变量
        this.state = { // 会渲染到视图上的状态
            width: 100,
            height: 100,
            left: 0,
            top: 0
        }
    }
}
```

那么问题来了，到底该用单个 state 变量还是多个 state 变量呢？
>* 如果使用单个 state 变量（对象的情况），<u>每次更新 state 时需要合并之前的 state</u>。因为 useState 返回的 setState 会替换原来的值。
这一点和 Class 组件的 this.setState 不同，this.setState 会把更新的字段自动合并到 this.state 对象中。
```
    /* class组件会把更新的字段自动合并到 this.state 对象 */
    this.setState({
        width: 200
    });

    /* 函数组件你可能需要这样写：*/
    setState({
        ...state,
        width: 200
    });

    /* 函数组件或者这样：*/
    setState(obj => ({...obj, width: 200}))
```

>* 但使用多个 state 变量的好处是可以让 state 的粒度更细，更易于逻辑的拆分和组合。
但如果粒度过细，代码就会变得比较冗余。如果粒度过粗，代码的可复用性就会降低。

总结下面两点:
 - 将完全不相关的 state 拆分为多组 state。比如 size 和 position。
 - 如果某些 state 是相互关联的，或者需要一起发生改变，就可以把它们合并为一组 state。比如 left 和 top。

**问题二：如何合理的使用useState的惰性初始化（函数）**

 - 如果 <u>初始 state 需要通过复杂计算获得</u> ，则可以传入一个函数，在函数中计算并返回初始的 state.
 - 此函数只在初始渲染时被调用.

```
const [state, setState] = useState(() => {
  const initialState = someExpensiveComputation(props);
  return initialState;
});
```

[例子还在考量中]

**问题三：函数式更新**
如果新的 state 需要通过使用先前的 state 计算得出，可以将函数传递给 setState。该函数将接收先前的 state，并返回一个更新后的值。

```
function Counter({initialCount}) {
  const [count, setCount] = useState(initialCount);
  return (
    <>
      Count: {count}
      <button onClick={() => setCount(initialCount)}>Reset</button>
      <button onClick={() => setCount(prevCount => prevCount - 1)}>-</button>
      <button onClick={() => setCount(prevCount => prevCount + 1)}>+</button>
    </>
  );
}
```

[实践场景一](https://codesandbox.io/s/muddy-sun-3r3qr?file=/src/App.js)
[实践场景二](https://codesandbox.io/s/usestatedehanshushigengxiner-r74n0?file=/src/App.js)