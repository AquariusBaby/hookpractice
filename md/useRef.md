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

---

**1. 字符串Refs（过时API）** [例子](https://codesandbox.io/s/jichongrefs-32vm3?file=/src/refStringDemo.js)

> **函数组件**内部**不支持**使用 字符串 refs [支持 createRef | useRef | 回调 Ref]

形如：
```javascript
class RefStringDemo extends React.Component {
  constructor(props) {
    super(props);
    this.state = {};
    this.onHandle = this.onHandle.bind(this);
  }

  // 需要在 this.refs 下访问对应的字符串
  onHandle() {
    this.refs.inputRef.focus();
  }

  render() {
    return (
      <div>
        // 以字符串的形式赋值
        <input ref={"inputRef"} />
        <button onClick={this.onHandle}>focus the input</button>
      </div>
    );
  }
}
```

 - ref用于DOM元素上，拿到的是DOM元素;
 - ref用于React组件上是什么样的：
    - 函数组件不行，因为没有实例；
    - 类组件可以，拿到的是 React.Component 类型

```javascript
console.log(this.refs.childRef, "看下ref用于函数组件实例上是什么样的"); // undefined
console.log(this.refs.childClassRef, "看下ref用于类组件实例上是什么样的"); // React.Component
```

---

**2. 回调Refs** [例子](https://codesandbox.io/s/jichongrefs-32vm3?file=/src/refCallbackDemo.js)

> 支持在函数组件和类组件内部使用

React 支持 回调 refs 的方式设置 Refs。**这种方式可以帮助我们更精细的控制何时 Refs 被设置和解除。**

使用 回调 refs 需要将回调函数传递给 React元素 的 ref 属性。这个函数接受 React 组件实例 或 HTML DOM 元素作为参数，将其挂载到实例属性上

形如：
```javascript
class RefCallbackDemo extends React.Component {
  constructor(props) {
    super(props);
    this.state = {};
    this.inputRef = null; // 定义一个实例属性inputRef来接受Ref
    this.onHandle = this.onHandle.bind(this);
  }

  onHandle() {
    this.inputRef.focus();
  }

  render() {
    return (
      <div>
        // 这样，回调接受一个 DOM元素 或 React组件实例 ；
        <input ref={(node) => (this.inputRef = node)} />
        <button onClick={this.onHandle}>focus the input</button>
      </div>
    );
  }
}
```

 - ref用于DOM元素上，拿到的是DOM元素;
 - ref用于React组件上是什么样的：
    - 函数组件不行，因为没有实例；
    - 类组件可以，拿到的是 React.Component 类型

```javascript
console.log(this.refs.childRef, "看下ref用于函数组件实例上是什么样的"); // null
console.log(this.refs.childClassRef, "看下ref用于类组件实例上是什么样的"); // React.Component
```

可以在组件间传递回调形式的 refs，如下：

```javascript
import React from 'react';

export default function Form() {
    let ref = null;
    React.useEffect(() => {
        //ref 即是 MyInput 中的 input 节点
        ref.focus();
    }, [ref]);

    return (
        <>
            <MyInput inputRef={ele => ref = ele} />
            {/** other code */}
        </>
    )
}

function MyInput (props) {
    return (
        <input type="text" ref={props.inputRef}/>
    )
}
```

---

**3. createRef** [例子]()

> 支持在函数组件和类组件内部使用

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

---

**4. useRef**

> 仅限于在函数组件内使用

 - React.useRef() 返回的 ref 对象在组件的整个生命周期内保持不变；
 - React.createRef() 每次都是重新赋值；

```javascript
import React, { useRef, useEffect, createRef, useState } from 'react';
function MyInput() {
    let [count, setCount] = useState(0);

    const myRef = createRef(null);
    const inputRef = useRef(null);
    //仅执行一次
    useEffect(() => {
        inputRef.current.focus();
        window.myRef = myRef;
        window.inputRef = inputRef;
    }, []);
    
    useEffect(() => {
        //除了第一次为true， 其它每次都是 false 【createRef】
        console.log('myRef === window.myRef', myRef === window.myRef);
        //始终为true 【useRef】
        console.log('inputRef === window.inputRef', inputRef === window.inputRef);
    })
    return (
        <>
            <input type="text" ref={inputRef}/>
            <button onClick={() => setCount(count+1)}>{count}</button>
        </>
    )
}
```

---

**5. Ref 的传递**

> 在 Hook 之前，高阶组件(HOC) 和 render props 是 React 中复用组件逻辑的主要手段。
尽管高阶组件的约定是将所有的 props 传递给被包装组件，但是 refs 是不会被传递的，事实上， **ref 并不是一个 prop，和 key 一样，它由 React 专门处理**。

所以在16.3版本新增了 React.forwardRef 来解决；

**在 React.forwardRef 之前**
```javascript
import React from 'react';
import hoistNonReactStatic from 'hoist-non-react-statics';

const withData = (WrappedComponent) => {
    class ProxyComponent extends React.Component {
        componentDidMount() {
            //code
        }
        //这里有个注意点就是使用时，我们需要知道这个组件是被包装之后的组件
        //将ref值传递给 forwardedRef 的 prop
        render() {
            const {forwardedRef, ...remainingProps} = this.props;
            return (
                <WrappedComponent ref={forwardedRef} {...remainingProps}/>
            )
        }
    }
    //指定 displayName.   未复制静态方法(重点不是为了讲 HOC)
    ProxyComponent.displayName = WrappedComponent.displayName || WrappedComponent.name || 'Component';
    //复制非 React 静态方法
    hoistNonReactStatic(ProxyComponent, WrappedComponent);
    return ProxyComponent;
}
```

这个示例中，我们将 ref 的属性值通过 forwardedRef 的 prop，传递给被包装的组件，使用：

```javascript
class MyInput extends React.Component {
    render() {
        return (
            <input type="text" {...this.props} />
        )
    }
}

MyInput = withData(MyInput);
function Form(props) {
    const inputRef = React.useRef(null);
    React.useEffect(() => {
        console.log(inputRef.current)
    })
    //我们在使用 MyInput 时，需要区分其是否是包装过的组件，以确定是指定 ref 还是 forwardedRef
    return (
        <MyInput forwardedRef={inputRef} />
    )
}
```

**在有了 React.forwardRef 之后**

Ref 转发是一项将 ref 自动地通过组件传递到其一子组件的技巧，其允许某些组件接收 ref，并将其向下传递给子组件。

```javascript
import React from 'react';

const MyInput = React.forwardRef((props, ref) => {
    return (
        <input type="text" ref={ref} {...props} />
    )
});
function Form() {
    const inputRef = React.useRef(null);
    React.useEffect(() => {
        console.log(inputRef.current);//input节点
    })
    return (
        <MyInput ref={inputRef} />
    )
}
```

---

**React.forwardRef**

React.forwardRef 会创建一个React组件，这个组件能够将其接受的 ref 属性转发到其组件树下的另一个组件中。这种技术并不常见，但在以下两种场景中特别有用：

 - [转发 refs 到 DOM 组件](https://zh-hans.reactjs.org/docs/forwarding-refs.html#forwarding-refs-to-dom-components)
 - [在高阶组件中转发 refs](https://zh-hans.reactjs.org/docs/forwarding-refs.html#forwarding-refs-in-higher-order-components)

React.forwardRef 接受渲染函数作为参数。React 将使用 props 和 ref 作为参数来调用此函数。此函数应返回 React 节点。

```javascript
const FancyButton = React.forwardRef((props, ref) => (
  <button ref={ref} className="FancyButton">
    {props.children}
  </button>
));

// You can now get a ref directly to the DOM button:
const ref = React.createRef();
<FancyButton ref={ref}>Click me!</FancyButton>;
```

**注意**

第二个参数 ref 只在使用 React.forwardRef 定义组件时存在。常规函数和 class 组件不接收 ref 参数，且 props 中也不存在 ref。

Ref 转发不仅限于 DOM 组件，你也可以转发 refs 到 class 组件实例中。