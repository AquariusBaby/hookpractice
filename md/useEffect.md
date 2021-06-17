#useEffect
---

**该 Hook 接收一个包含命令式、且可能有副作用代码的函数。**

在函数组件主体内（这里指在 React 渲染阶段）改变 DOM、添加订阅、设置定时器、记录日志以及执行其他包含 <u>副作用</u> 的操作都是不被允许的，因为这可能会产生莫名其妙的 bug 并破坏 UI 的一致性。

使用 useEffect 完成副作用操作。赋值给 useEffect 的函数会在组件渲染到屏幕之后执行。你可以把 effect 看作从 React 的纯函数式世界通往命令式世界的逃生通道

```
useEffect(() => { 
    // 执行effect
    ...
    return () => {
        // 清除effect
        ...
    }
}, [...deps]);
```

**问题一：useEffect回调的执行的时机**

A：useEffect 的函数会在组件渲染到屏幕之后执行。

也就是传给 useEffect 的函数会在浏览器完成布局与绘制之后，在一个延迟事件中被调用。这使得它适用于许多常见的副作用场景，比如设置订阅和事件处理等情况，因为绝大多数操作不应阻塞浏览器对屏幕的更新。

1. 并非所有 effect 都可以被延迟执行，可以参考 useEffectLayout；
2. 虽然 useEffect 会在浏览器绘制后延迟执行，但会保证在任何新的渲染前执行；
3. 和生命周期componentDidXXX的执行时机是不同，也不是一档子事，你应该抛弃掉用useEffect去模拟生命周期的想法。当然真要这样做，你可以参考[ahooks提供的自定义hook](https://ahooks.js.org/zh-CN/hooks/life-cycle/use-mount)；

**问题二：useEffect清除函数的执行时机**

A：上一次的更新的返回清理函数，会在下一次更新渲染完之后 和 下一次更新渲染完的effect回调执行之前 执行清除。

> 假设第一次渲染的时候props是{id: 10}，第二次渲染的时候是{id: 20}。
那么执行流程是这样的：
 - React 渲染{id: 10}的UI
 - 浏览器绘制。我们在屏幕上看到{id: 10}的UI
 - React 运行{id: 10}的effect
 - React 渲染{id: 20}的UI
 - 浏览器绘制。我们在屏幕上看到{id: 20}的UI
 - **React 清除{id: 10}的effect**
 - React 运行{id: 20}的effect

1. 为防止内存泄漏，清除函数会在组件卸载前执行；
2. 所有的清除函数会在下一次更新的effect执行之前，全部执行完；

**问题三：useEffect的依赖是否可以不按要求来**

A：永远不要欺骗useEffect，你用到了什么状态、或者函数就应该添加到依赖中去；

推荐启用 eslint-plugin-react-hooks 中的 exhaustive-deps 规则。此规则会在添加错误依赖时发出警告并给出修复建议。

**问题四：useEffect中出现的无限重复请求的问题**

A：其实就是依赖项包含的状态，你在useEffect内又去执行了该状态的状态更新函数，这就导致了一个无限循环的情况。

可以看下：[实践场景一](https://codesandbox.io/s/useeffectwuxianqingqiuchangjing-zl9tk?file=/src/App.js)

实践场景一使用的解决方案：
 - 利用useState的函数式更新;
 - 使用useReducer，因为React会保证 dispatch 的稳定性;

**问题五：deps 依赖过多，导致 Hooks 难以维护？（其实这个不算是问题）**

使用 useEffect hook 时，为了避免每次 render 都去执行它的 callback，我们通常会传入第二个参数「dependency array」
```
useEffect(() => {
  // ...
}, [name, searchState, address, status, personA, personB, progress, page, size]);
```

- 你需要重新思考一下，这些 deps 是否真的都需要？(基本都是需要的)
- 如果这些依赖真的都是需要的，那么这些逻辑是否应该放到同一个 hook 中？
（如果里面有些逻辑是相互独立的，我们可以将独立逻辑放到不同 useEffect 中，这样依赖有可能也会被拆分出来）
- 加入依赖本身就很多，可以考虑把这些依赖看做一个整体，如下：
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

我们也来尝试看一下一个常见的[场景二](https://codesandbox.io/s/useeffectdeguoduoyilai-xjt7e)

<!-- **问题六：合并多个useEffect的情况** -->
**问题六：应该把函数当做effect的依赖吗？**

 - 如果你的函数中不依赖state和props，一般建议可以把这个函数提到组件外面
 - 如果你的函数依赖props或state，你应该把这个函数放到useEffect中
 - 如果你的函数依赖props或state，但你又不想放到useEffect中，可以使用useCallback包起来（通常这种情况是你需要复用这个函数）

**我们来看下这三种情况**：[场景三](https://codesandbox.io/s/useeffectzhongdehanshu-gqsmc?file=/src/App.js)

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

**问题七：useEffect中的使用async await**

```
// 一个错误的写法

useEffect(async () => {
  await ....
}, []);

```

Effect回调是同步的，以防止出现竞争条件。得把async函数放在里面

```
// 正确的写法
useEffect(() => {
  async function fetchData() {
    const result = await axios(getFetchUrl());
    setData(result.data);
  }
}, []);
```

**问题八：React竞态问题**

下面是一个典型的在class组件里发请求的例子：
```
class Article extends Component {
  state = {
    article: null
  };
  componentDidMount() {
    this.fetchData(this.props.id);
  }
  async fetchData(id) {
    const article = await API.fetchArticle(id);
    this.setState({ article });
  }
  // ...
}
```

上面的代码埋伏了一些问题。**它并没有处理更新的情况**。所以第二个你能够在网上找到的经典例子是下面这样的：

```
class Article extends Component {
  state = {
    article: null
  };
  componentDidMount() {
    this.fetchData(this.props.id);
  }
  componentDidUpdate(prevProps) {
    if (prevProps.id !== this.props.id) {
      this.fetchData(this.props.id);
    }
  }
  async fetchData(id) {
    const article = await API.fetchArticle(id);
    this.setState({ article });
  }
  // ...
}
```

这显然好多了！但依旧有问题。有问题的原因是请求结果返回的顺序不能保证一致。比如我先请求 {id: 10}，然后更新到{id: 20}，但{id: 20}的请求更先返回。请求更早但返回更晚的情况会错误地覆盖状态值。

这被叫做**竞态**，这在混合了async / await（假设在等待结果返回）和自顶向下数据流的代码中非常典型（props和state可能会在async函数调用过程中发生改变）。

如果你使用的异步方式支持取消，那太棒了。你可以直接在清除函数中取消异步请求。

或者，最简单的权宜之计是**用一个布尔值来跟踪它**：

```
function Article({ id }) {
  const [article, setArticle] = useState(null);

  useEffect(() => {
    let didCancel = false;

    async function fetchData() {
      const article = await API.fetchArticle(id);
      if (!didCancel) {
        setArticle(article);
      }
    }

    fetchData();

    return () => {
      didCancel = true;
    };
  }, [id]);

  // ...
}
```

**问题九：Suspense**
有待补充中...