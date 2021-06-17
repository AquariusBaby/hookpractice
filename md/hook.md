# React hook
------
 1. what is hook？
 2. why is hook?
 3. how?

------
##什么是hook？
**React Hooks** 是一些可以让你在函数组件里“钩入” React state 及生命周期等特性的函数
**React Hooks** 是在React函数组件中使用状态和副作用的一种方法。

### 如何存储状态
函数组件是不具备保存和维护自己状态能力的，通过hook则可以实现在函数组件中勾入状态。如何实现的？这个状态保存在哪？

用一个例子来看下：[例子二](https://codesandbox.io/s/liziyi-n1zwy?file=/src/App.js)

```
function App() {
  const [count, setCount] = useState(0); // hook1
  const [num, setNum] = useState(0); // hook2
  
  //...
}

// 看下上面的App对应的fiber数据结构大致如下：
var fiber = {
  // 指向App函数本身
  stateNode: App,
  // 保存该FunctionComponent对应的Hooks链表，单向的无环的(不闭合的)的链表
  memoizedState: hook1 -> hook2 -> null
};

// 再看下每个hook对应的数据结构大致如下：
var hook1 = {
  // 保存update的queue，单向环状链表
  queue: {
    pending: null
  },
  memoizedState: initialState, // 初始值0
  next: hook2 // 链接下一个hook
};
```
每个hook的值就保存在当前函数组件Fiber下的hook链表中，且一一对应，状态页得以存储。

### 再来看看hook是怎么运作的
> 先来看下React架构：
React16版本是 **Scheduler-Reconciler-Renderer** 架构体系
**Reconciler**工作的阶段被称为**render阶段**。因为在该阶段会调用组件的render方法。
**Renderer**工作的阶段被称为**commit阶段**。会把render阶段提交的信息渲染在页面上。
**render**与**commit**阶段统称为work，即React在工作中，任务正在**Scheduler**内调度，就不属于work。

 1. **Schedule**：当我们触发一次状态更新操作，react会从当前触发更新的Fiber节点向上找到根节点，即rootFiber，接下来通知Scheduler根据更新的优先级，看以同步还是异步的方式去调度本次更新。(先不关心调度的优先级概念)

```
  // 开启调度
  scheduleCallback(
    schedulerPriorityLevel, // 某个优先级
    performConcurrentWorkOnRoot.bind(null, root) // 以concurrent模式
  )
```
performConcurrentWorkOnRoot会进入到 **Reconciler** 也就是**render阶段**。

**render阶段**包含两个部分：beginWork、completeWork

 - beginWork会去执行Fiber.stateNode()，即App();
 - 这时候进入到App()的执行中，分为两种情况mount、update;
 
```
// App对应的fiber是这样的：
fiberApp: {
    memoizedState: null
    // ...
}

// 执行App()，当遇到useState()；我们来看下useState；
function App() {
    let wipHook; // 指向正在工作的hook
    
    const [count, setCount] = useState(1); // hook1
    const [num, setNum] = useState(2); // hook2
    // .....
    return React.createElement(jsx);
}

// isMount用来模拟mount和update（真实的hook不是这样区分的）、wipHook指向当前正在执行的hook的指针；
function useState(init) {
    let hook;
    // 第一部分：hook的初始化
    // 第二部分：hook的更新
    // 第三部分：hook如何返回 [值, 更新函数]
}
```

**第一部分：hook的初始化**
```
// 初始化时，fiber.memoizedState上的hook链表是null
if (isMount) {
    // 初始化hook
    hook = { 
      queue: {pending: null},
      memoizedState: init,
      next: null
    }
    // 将hook插入到fiber.memoizedState链表
    if (!fiber.memoizedState) {
      fiber.memoizedState = hook;
    } else {
      wipHook.next = hook;
    }
    // 移动wipHook指针
    wipHook = hook;
    
    // 把结果返回，dispatchAction方法是更新方法
    return [hook.memoizedState, dispatchAction.bind(null,hook.queue, action)
}
```

不断创建hook并初始化hook.memoiezd = init，初始化后结果如下：
```
fiberApp: {
    memoizedState: hook1 -> hook2 -> .... -> null
    // ...
}

hook1: {
  queue: {
    pending: null
  },
  memoizedState: 1, // 初始值
  next: hook2 // 链接下一个hook
}
hook2: {
  queue: {
    pending: null
  },
  memoizedState: 2, // 初始值
  next: null // 链接下一个hook
}
```

**第二部分：hook的更新**
> 调用setCount，也就是dispacthAction.bind(hook.queue, action)，当前hook对应的存在更新的队列被绑定进去了；
setCount(2) 等价于 dispatchAction(2)；
dispatchAction会把2的update对象加入到队列中；开启调度进入Schedule，这里我们再次进入到调度阶段，只不过这次是触发更新，不是初始化;

> 触发更新 -> Scheduler调度 -> Resconciler协调(render阶段) -> benginWork中去再次执行fiber.stateNode(), 即执行App()，再次遇到useState; 


```
function useState() {
    // 更新时，fiber.memoizedState已经存在一个按声明顺序连接的链表
    if(!isMount) {
        // 赋值hook为对应的wipHook
        hook = wipHook;
        // 移动wipHook指针
        wipHook = wipHook.next;
        
        let baseState = hook.memoizedState; //拿到上一次更新后的值
        
        // 如果当前hook的queue不为空，即存在update，则更新其state
        if (hook.queue.pending) { 
            // 拿到第一个update
            let firstUpdate = hook.queue.pending.next;
            // 遍历当前hook下的queue，计算最终的state;
            do {
              // 执行update的action
              const action = firstUpdate.action;
              // 这里的action可能函数或者常量
              baseState = action(baseState);
              firstUpdate = firstUpdate.next;
            } while (firstUpdate !== hook.queue.pending.next)
        
            // 更新完，清空queue.pending
            hook.queue.pending = null;
        
            // 赋值更新后state
            hook.memoizedState = baseState;
        }
        
        return [hook.memoizedState, dispatchAction.bind(null,hook.queue, action)
    }
}
```

现在我们拿到计算后的state，App()最后一步是return React.createElement(jsx)，返回我结果会与old fiber进行diff，比对出差异部分，并给当前fiber打上effectTag副作用标记也就是增、删、改的标记，并将当前fiber添加到rootFiber的effectList副作用链表上；

待**Reconciler**完成，进入到**Renderer**，也就是**commit阶段**，到这里就是同步遍历rootFiber的effectList副作用链表，执行相应的DOM操作，然后渲染视图；

commit阶段包含三个部分：before mutation(执行DOM操作前)、mutation(执行DOM操作)、layout(执行DOM操作后)；我们看下这三个阶段跟hook的关系:

 1. before mutation(执行DOM操作前)：这里会执行getSnapshotBeforeUpdate的生命周期钩子；
 2. mutation(执行DOM操作)：这里执行DOM的操作、执行useLayoutEffect的销毁函数；
 3. layout(执行DOM操作后)：
    >* 执行componentDidXXX的生命周期；
    >* 执行setState(_, cb)的第二个回调函数；
    >* 执行ReactDom.render(_, _ , cb)的第三回调函数；
    >* 执行useLayoutEffect的回调；
    >* 异步调度useEffect的销毁函数；
    >* 异步调度useEffect的回调;
```
scheduleCallback(NormalSchedulerPriority, () => {  
    // NormalSchedulerPriority普通优先级的
    // 触发useEffect
    flushPassiveEffects(); // 要去拿effectList去遍历执行
    return null;
});
```

------
##hook的动机？
在组件之间复用状态逻辑很难，复杂组件变得难以理解等等；

**React Hooks** 要解决的问题是状态共享，是继 render-props 和 higher-order components 之后的第三种状态共享方案，不会产生 JSX 嵌套地狱问题。

> 这个状态指的是状态逻辑，所以称为状态逻辑复用会更恰当，因为只共享数据处理逻辑，不会共享数据本身

用一个例子来体验一下：[例子一](https://codesandbox.io/s/zhuangtailuojifuyongliziyi-civpe?file=/src/App.js)

可以看到，React Hooks 就像一个内置的打平 renderProps 库，我们可以随时创建一个值，与修改这个值的方法。

hook并不是来替代HOC、render props的，hook应该说是提供了一种颗粒度更小的状态逻辑共享方式；

------
##如何使用Hook?

**Hook 使用规则**
> 只能在函数最外层调用 Hook。不要在循环、条件判断或者子函数中调用。
只能在 React 的函数组件中调用 Hook。不要在其他 JavaScript 函数中调用。(自定义hook除外)

 1. const [state, update] = useState(initVal);

  - 接受一个值或者函数，来初始化state；返回一个值及其对应的更新函数；
  - 惰性初始 state：如果初始 state 需要通过复杂计算获得，则可以传入一个函数，在函数中计算并返回初始的 state。（用于初始化的值和函数仅在初始渲染时会被调用）
  - 跳过 state 更新： 调用update更新函数并传入当前的 state 时，React 将跳过子组件的渲染及 effect 的执行。（React 使用 Object.is 比较算法 来比较 state，相同时会被跳过）；
  - update具有稳定性：React 会确保 update 函数的标识是稳定的，并且不会在组件重新渲染时发生变化，你可以从 useEffect 或 useCallback 的依赖列表中省略 update
  - update接受一个更新值或者一个函数，函数的入参为上一次的旧值；

  2. const [state, dispatch] = useReducer(reducer, initVal, initFn);
  - 接受一个纯函数reducer、初始值initVal、惰性初始化函数；
  - dispatch具有稳定性：同意React 会确保 dispatch 函数的标识是稳定的；

  3. useEffect
  ```
  useEffect(() => { 
      // 执行effect
      return () => {
          // 清除effect
      }
  }, [...deps]);
  ```

---
在我们讨论effects之前，我们需要先讨论一下渲染（rendering）。
1. 每一次渲染都有它自己的 Props and State
```
// 看下这个
function Counter() {
  const [count, setCount] = useState(0);

  return (
    <div>
      <p>You clicked {count} times</p>
      <button onClick={() => setCount(count + 1)}>
        Click me
      </button>
    </div>
  );
}
```
它没有神奇的“data binding”, “watcher”, “proxy”，或者其他任何东西。它就是一个普通的数字像下面这个一样：
```
const count = 42;
// ...
<p>You clicked {count} times</p>
// ...
```

当我们更新状态的时候，React会重新渲染组件。每一次渲染都能拿到独立的count 状态，这个状态值是函数中的一个常量。

2.每一次渲染都有它自己的事件处理函数
[来看个例子](https://codesandbox.io/s/w2wxl3yo0l?file=/src/index.js)，并按如下步骤操作：

- 点击增加counter到3
- 点击一下 “Show alert”
- 点击增加 counter到5并且在定时器回调触发前完成

结果：alert的结果却是3；

> 实际上，每一次渲染都有一个“新版本”的handleAlertClick。每一个版本的handleAlertClick“记住” 了它自己的 count：

3.每次渲染都有它自己的Effects
[来看例子](https://codesandbox.io/s/hooklizi1-6b2uo?file=/src/App.js)

打印结果是：
You clicked 1 times 
You clicked 2 times 
You clicked 3 times 
You clicked 4 times 
You clicked 5 times 

> 同理上面的函数，没诚意渲染都有一个“新版本”的useEffect，每一个effect版本“看到”的count值都来自于它属于的那次渲染

看到这里，你可能会想：“它当然应该是这样的。否则还会怎么样呢？”
我们来看下[class版本](https://codesandbox.io/s/kkymzwjqz3)的实现 

```
componentDidUpdate() {
    setTimeout(() => {
      console.log(`You clicked ${this.state.count} times`);
    }, 3000);
}
```

打印结果：
You clicked 5 times
You clicked 5 times 
You clicked 5 times 
You clicked 5 times 
You clicked 5 times 

>* 因为，this.state.count总是指向最新的count值，而不是属于某次特定渲染的值。所以你会看到每次打印输出都是5；

>* 场景实例: 有一个类似 twitter 的页面, 你想要 follow 某个用户, 但是点击 follow 这个动作是一个异步请求可能需要时间, 在点击了 follow 这个操作之后, 立马切换到另一个用户的页面, 几秒钟之后客户端收到响应, 显示你 follow 了"这个"用户[例子](https://codesandbox.io/s/pjqnl16lm7?file=/src/index.js)

>* class 组件之所以有时候"不太对"的原因是, React 修改了 class 中的 this.state 使其指向永远最新状态


4. 函数式组件在每一次渲染都有它自己的…所有,
你可以想象成每次 render 的时候都形成了一次快照, 保存了所有下面的东西, 每一份快照都是不同且独立的. 即
每一次渲染都有自己的 props 和 state
每一次渲染都有自己的事件处理函数
每一次渲染都有自己的 useEffect()
class 组件之所以有时候"不太对"的原因是, React 修改了 class 中的 this.state 使其指向永远最新状态

----
关于依赖项不要对React撒谎：我们经常会遇到在useEffect里面出现无限请求的问题；
```
// ...
const [list, update] = useState([]);
useEffect(() => {
    // ...发起异步请求
    (async function() {
        let response = await API.fetch();
        update([...list, ...response]);
    })()
    // ...
}, [list]);
// ... 
```

依赖是正确的，没有问题，但会出现无限请求
这个时候，我嗯可以利用Hooks的作弊模式useReducer

```
function reducer(state, action) {
    switch(action.type) {
        case 'add':
            return [...state, ...action.payload]
        default:
            return state;
    }
}

const [list, dispatch] = useReducer(reducer, []);

useEffect(() => {
    // ...发起异步请求
    (async function() {
        let response = await API.fetch();

        // update([...list, ...response]);
        dispatch({
            type: 'add',
            payload: response
        });
    })()
    // ...
}, [dispatch]);

// 因为React会保证dispatch的稳定性，所以不出出现无限请求
```
---

让Effects自给自足
```
useEffect(() => {
    const id = setInterval(() => {
      setCount(count + 1);
    }, 1000);
    return () => clearInterval(id);
}, [count]);
```
这个例子，我们想去掉effect的count依赖。
因为每次setCount，都会导致useEffect的重新执行，而我们设置的定时器就会不断的被新增，然后在被清除，是似乎并不是我们想要的。

在这个场景中，我们其实并不需要在effect中使用count。当我们想要根据前一个状态更新状态的时候，我们可以使用setState的函数形式：
```
useEffect(() => {
    const id = setInterval(() => {
      setCount(c => c+1);
    }, 1000);
    return () => clearInterval(id);
}, []);

// 注意我们做到了移除依赖，并且没有撒谎
```

---
把函数移到Effects里
一个典型的误解是认为函数不应该成为依赖
```
function SearchResults() {
  const [data, setData] = useState({ hits: [] });

  async function fetchData() {
    const result = await axios(
      'https://hn.algolia.com/api/v1/search?query=react',
    );
    setData(result.data);
  }

  useEffect(() => {
    fetchData();
  }, []); // Is this okay?

  // ...
}
```
[例子](https://codesandbox.io/s/quizzical-archimedes-ei7sn?file=/src/App.js)

解决方案：可以把它们的定义移到effect中
```
function SearchResults() {
  // ...
  useEffect(() => {
    // We moved these functions inside!
    function getFetchUrl() {
      return 'https://hn.algolia.com/api/v1/search?query=react';
    }
    async function fetchData() {
      const result = await axios(getFetchUrl());
      setData(result.data);
    }

    fetchData();
  }, []); // ✅ Deps are OK
  // ...
}
```
> 这么做有什么好处呢？我们不再需要去考虑这些“间接依赖”。我们的依赖数组也不再撒谎：在我们的effect中确实没有再使用组件范围内的任何东西。

---
但我不能把这个函数放到Effect里
有时候你可能不想把函数移入effect里。比如，组件内有几个effect使用了相同的函数，你不想在每个effect里复制黏贴一遍这个逻辑。
```
function SearchResults({ query }) {
  function getFetchUrl() {
    return 'https://hn.algolia.com/api/v1/search?query=' + query;
  }

  useEffect(() => {
    const url = getFetchUrl('react');
    // ... Fetch data and do something ...
  }, [getFetchUrl]);

  useEffect(() => {
    const url = getFetchUrl('redux');
    // ... Fetch data and do something ...
  }, [getFetchUrl]);

  // ...
}
```

我们的两个effects都依赖getFetchUrl，而它每次渲染都不同，所以我们的依赖数组会变得无用；这个时候我们可以考虑使用useCallback

```
function SearchResults({ query }) {
  // ✅ Preserves identity when its own deps are the same
  const getFetchUrl = useCallback(() => {
    return 'https://hn.algolia.com/api/v1/search?query=' + query;
  }, [query]);  // ✅ Callback deps are OK

  useEffect(() => {
    const url = getFetchUrl('react');
    // ... Fetch data and do something ...
  }, [getFetchUrl]); // ✅ Effect deps are OK

  useEffect(() => {
    const url = getFetchUrl('redux');
    // ... Fetch data and do something ...
  }, [getFetchUrl]); // ✅ Effect deps are OK

  // ...
}
```

> useCallback本质上是添加了一层依赖检查。它以另一种方式解决了问题 - 我们使函数本身只在需要的时候才改变，而不是去掉对函数的依赖。

useCallback当然不仅仅只是这一个用处，当我们遇到：
```
function Parent() {
    // ...
    function update() {
        // ...
    }

    return (
        <>
            // ...
            <Child update={update} />
        </>
    )
}
```
上面已经说过了，每次渲染都有一个“新版本”的函数，所以这个update函数属于引用类型，每次都不同，所以每次Parent的渲染都会导致Child的渲染，处理如下：
```
const update = useCallback(() => {
    // ....
}, [])
```

同理的还有useMemo，通常用于对象
```
const memoizedValue = useMemo(() => {
    // 某个昂贵的复杂的计算
    computeExpensiveValue(a, b);
}, [a, b]);
```

----
最后我们来根据一些问题来总结一些实践上的写法：

**问题一：我该使用单个 state 变量还是多个 state 变量？**
useState 的出现，让我们可以使用多个 state 变量来保存 state，比如：
```
const [width, setWidth] = useState(100);
const [height, setHeight] = useState(100);
const [left, setLeft] = useState(0);
const [top, setTop] = useState(0);
```
同样，我们可以将所有的 state 放到一个 object 中
```
const [state, setState] = useState({
  width: 100,
  height: 100,
  left: 0,
  top: 0
});
```

那么问题来了，到底该用单个 state 变量还是多个 state 变量呢？
>* 如果使用单个 state 变量，每次更新 state 时需要合并之前的 state。因为 useState 返回的 setState 会替换原来的值。这一点和 Class 组件的 this.setState 不同。this.setState 会把更新的字段自动合并到 this.state 对象中。

>* 但使用多个 state 变量的好处是可以让 state 的粒度更细，更易于逻辑的拆分和组合。
但如果粒度过细，代码就会变得比较冗余。如果粒度过粗，代码的可复用性就会降低。

总结下面两点:
 - 将完全不相关的 state 拆分为多组 state。比如 size 和 position。
 - 如果某些 state 是相互关联的，或者需要一起发生改变，就可以把它们合并为一组 state。比如 left 和 top。

**问题二：deps 依赖过多，导致 Hooks 难以维护？**
使用 useEffect hook 时，为了避免每次 render 都去执行它的 callback，我们通常会传入第二个参数「dependency array」
```
const refresh = useCallback(() => {
  // ...
}, [name, searchState, address, status, personA, personB, progress, page, size]);
```

- 你需要重新思考一下，这些 deps 是否真的都需要？(大多情况都是需要的)
- 如果这些依赖真的都是需要的，那么这些逻辑是否应该放到同一个 hook 中？
（如果里面有些逻辑是相互独立的，我们可以将独立逻辑放到不同 useEffect 中，这样依赖有可能也会被拆分出来）
- 加入依赖本身就很多，我嗯可以考虑把这些依赖看做一个整体，如下：
```
const [filters, setFilters] = useState({
    name: "",
    address: "",
    status: "",
    personA: "",
    personB: "",
    progress: ""
});

useEffect(() => {
    id && doSearch(filters);
}, [id, filters]);
  ```

**问题三：该不该使用 useMemo？**
我们常用 useMemo 或者 useCallback 「包裹一下」来做一些优化（减少一些不必要的渲染），似乎就能使应用远离性能的问题。但真的是这样吗？有的时候 useMemo 没有任何作用，甚至还会影响应用的性能。

> 首先，useMemo本身也有开销。useMemo 会「记住」一些值，同时在后续 render 时，将依赖数组中的值取出来和上一次记录的值进行比较，如果不相等才会重新执行回调函数，否则直接返回「记住」的值。这个过程本身就会消耗一定的内存和计算资源。因此，过度使用 useMemo 可能会影响程序的性能。

要想合理使用 useMemo，我们需要搞清楚 useMemo 适用的场景：
- 有些计算开销很大，我们就需要「记住」它的返回值，避免每次 render 都去重新计算。
- 由于值的引用发生变化，导致下游组件重新渲染，我们也需要「记住」这个值。

```
interface IExampleProps {
  page: number;
  type: string;
}

const Example = ({page, type}: IExampleProps) => {
  const resolvedValue = useMemo(() => {
    return getResolvedValue(page, type);
  }, [page, type]);

  return <ExpensiveComponent resolvedValue={resolvedValue}/>;
};
```

> 在上面的例子中，假设 ExpensiveComponent 是一个渲染开销很大的组件。所以，当 resolvedValue 的引用发生变化时，作者不想重新渲染这个组件。因此，作者使用了 useMemo，避免每次 render 重新计算 resolvedValue，导致它的引用发生改变，从而使下游组件 re-render。

这个担忧是正确的，但是使用 useMemo 之前，我们应该先思考两个问题：
1. 传递给 useMemo 的函数开销大不大？在上面的例子中，就是考虑 getResolvedValue 函数的开销大不大。JS 中大多数方法都是优化过的，比如 Array.map、Array.forEach 等。如果你执行的操作开销不大，那么就不需要记住返回值。否则，使用 useMemo 本身的开销就可能超过重新计算这个值的开销。因此，对于一些简单的 JS 运算来说，我们不需要使用 useMemo 来「记住」它的返回值。
2. 当输入相同时，「记忆」值的引用是否会发生改变？在上面的例子中，就是当 page 和 type 相同时，resolvedValue 的引用是否会发生改变？这里我们就需要考虑 resolvedValue 的类型了。如果 resolvedValue 是一个对象，由于我们项目上使用「函数式编程」，每次函数调用都会产生一个新的引用。但是，如果 resolvedValue 是一个原始值（string, boolean, null, undefined, number, symbol），也就不存在「引用」的概念了，每次计算出来的这个值一定是相等的。也就是说，ExpensiveComponent 组件不会被重新渲染。

> 因此，如果 getResolvedValue 的开销不大，并且 resolvedValue 返回一个字符串之类的原始值。或者ExpensiveComponent渲染的成本很小。那我们完全可以去掉 useMemo。

> 还有一个误区就是对创建函数开销的评估。有的人觉得在 render 中创建函数可能会开销比较大，为了避免函数多次创建，使用了 useMemo 或者 useCallback。但是对于现代浏览器来说，创建函数的成本微乎其微。

**问题四：Hooks 能替代高阶组件和 Render Props 吗？**
上面的React Hook动机已经提到过了，直接看结论：
>* 官方给出的回答是，在高阶组件或者 Render Props 只渲染一个子组件时，Hook 提供了一种更简单的方式。不过在我看来，Hooks 并不能完全替代 Render Props 和高阶组件。也就是说Hooks 并不能完全替代 Render Props 和高阶组件。

>* 没有 Hooks 之前，高阶组件和 Render Props 本质上都是将复用逻辑提升到父组件中。而 Hooks 出现之后，我们将复用逻辑提取到组件顶层，而不是强行提升到父组件中。这样就能够避免 HOC 和 Render Props 带来的「嵌套地域」。但是，像 Context 的 <Provider/> 和 <Consumer/> 这样有父子层级关系（树状结构关系）的，还是只能使用 Render Props 或者 HOC。

对于 Hooks、Render Props 和高阶组件来说，它们都有各自的使用场景：

- Hooks：
    - 替代 Class 的大部分用例，除了 getSnapshotBeforeUpdate 和 componentDidCatch 还不支持。
    - 提取复用逻辑。除了有明确父子关系的，其他场景都可以使用 Hooks。
- Render Props：在组件渲染上拥有更高的自由度，可以根据父组件提供的数据进行动态渲染。适合有明确父子关系的场景。
- 高阶组件：适合用来做注入，并且生成一个新的可复用组件。适合用来写插件。

>* 在大部分情况下，高阶组件和 Render Props 是可以相互转换的，也就是说用高阶组件能实现的，用 Render Props 也能实现。

**问题五： 使用 Hooks 时还有哪些好的实践？**
1. 若 Hook 类型相同，且依赖数组一致时，应该合并成一个 Hook。否则会产生更多开销。
2. 在使用 useMemo 或者 useCallback 时，确保返回的函数只创建一次。也就是说，函数不会根据依赖数组的变化而二次创建。
3. 自定义hook：我们以一个useImmer为例

>* 封装原本对 setState 增强的库
我们经常会说到React中的immutable数据不可变的概念，目前我觉得比较好用的库如immer.js,

immer 的语法是通过 produce 包装，将 mutable 代码通过 Proxy 代理为 immutable：

```
const nextState = produce(baseState, draftState => {
    draftState.push({ todo: "Tweet about it" });
    draftState[1].done = true;
})
```

但是你会发现这样写很繁琐，不够简洁、直观、优雅；尤其是这个produce这个方法

我可以通过自定义hook useImmer来 将 produce 来隐藏掉：[useImmer](https://codesandbox.io/s/custom-hooks-rb3v1?file=/src/hooks/useImmer.js)

```
function useImmer(initVal) {
  const [state, setState] = useState(initVal);

  function update(fn) {
    setState(produce(fn));
  }

  return [state, update];
}
```

使用方式就变成这样了：
```
const [value, setValue] = useImmer({ a: 1 });

value(obj => (obj.a = 2)); // immutable
```
是不是就更简洁、直观了，还不用担心改变了数据；