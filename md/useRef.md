# useRef
---

接受一个 <u>初始值</u>（这个初始值将被作为 useRef 返回值的current属性的值），返回一个 <u>可变的ref对象</u>，且这个ref对象在组件的整个生命周期内保持不变。ref对象的值发生改变之后，不会触发组件重新渲染。
```javascript
const refContainer = useRef(initialValue); // refContainer.current = initialValue
```

用法一：[常用于访问DOM](https://codesandbox.io/s/useref-zl8iy?file=/src/inputFocus.js)
```javascript
function TextInputWithFocusButton() {
    const inputEl = useRef(null);

    const onButtonClick = () => {
        inputEl.current.focus();
    }

    return (
        <>
            <input ref={inputEl} type="text" />
            <button onClick={onButtonClick}>Focus the input</button>
        <>
    )
}
```

用法二：[保存可变值](https://codesandbox.io/s/useref-zl8iy?file=/src/intervalTimer.js)（其类似于在 class 中使用实例字段的方式）
```javascript
function Timer() {
  const [count, setCount] = useState(0);
  const intervalRef = useRef();

  useEffect(() => {
    intervalRef.current = setInterval(() => {
      setCount((c) => c + 1);
    }, 1000);

    return () => {
      clearInterval(intervalRef.current);
    };
  }, []);

  return <div>{count}</div>;
}
```

如果我们只是想设定一个循环定时器，我们不会需要这个 ref（id 可以是在 effect 本地的），但如果我们想要在一个事件处理器中清除这个循环定时器的话这就很有用了：
```javascript
function Timer() {
  const [count, setCount] = useState(0);
  const intervalRef = useRef();

  useEffect(() => {
    intervalRef.current = setInterval(() => {
      setCount((c) => c + 1);
    }, 1000);

    return () => {
      clearInterval(intervalRef.current);
    };
  }, []);

  function clear() {
    clearInterval(intervalRef.current);
  }

  return (
    <>
      <div>{count}</div>
      <button onClick={() => clear()}>清楚定时器</button>
    </>
  );
}
```

---

**看到作用一的例子，我来看下之前refs的用法：**

<!-- 1. refs
2. createRef
3. useRef
4. forwardRef
5. useImperativeHandle -->

在某些情况下，我们需要在典型数据流之外强制修改子组件，被修改的子组件，可能是一个 **React 组件的实例**，也可能是一个 **DOM 元素**，对于这两种情况，React 都提供了解决办法。

下面是几个适合使用 refs 的情况：
 - 管理焦点，文本选择 或 媒体播放
 - 触发强制动画
 - 集成第三方DOM库

> 避免使用 refs 来做任何可以通过声明式实现来完成的事情。
举个例子，避免在 Dialog 组件里暴露 open() 和 close() 方法，最好传递 isOpen 属性。



1. createRef

形如：
```javascript
class component extends React.Component {
    constructor(props) {
        super(props);
        this.myRef = React.createRef();
    }

    componentDidMount() {
        // 当 ref 被传递给 render 中的元素时，对该节点的引用可以在 ref 的 current 属性中被访问。
        const node = this.myRef.current;
    }

    render() {
        return <div ref={this.myRef}></div>
    }
}
```

ref 的值根据节点的类型而有所不同：
 - 当 ref 属性用于 HTML 元素时，构造函数中使用 React.createRef() 创建的 ref 接收底层 DOM 元素作为其 current 属性。
 - 当 ref 属性用于自定义 class 组件时，ref 对象接收组件的挂载实例作为其 current 属性。
 - **你不能在函数组件上使用 ref 属性**，因为他们没有实例。


> React 会在组件挂载时给 current 属性传入 DOM 元素，并在组件卸载时传入 null 值。ref 会在 componentDidMount 或 componentDidUpdate 生命周期钩子触发前更新。至于UNSAFE_componentWillMount 时refs.current 为 null；


