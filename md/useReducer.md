# useReducer
---

接受一个形如 (state, action) => newState 的reducer、以及初始值、以及一个接受初始值为入参的惰性初始化函数。并返回当前的<u>state</u>和配套的<u>dispatch</u>方法（这个dispatch函数具有稳定性）。
```
const [state, dispatch] = useReducer(reducer, initialArg, init);
```

在此之前，你可以先看下[什么是reducer](https://github.com/AquariusBaby/hookpractice/blob/main/md/reducer.md)

在某些场景下，useReducer 会比 useState 更适用，例如 state 逻辑较复杂且包含多个子值，或者下一个 state 依赖于之前的 state 等。并且，**使用 useReducer 还能给那些会触发深更新的组件做性能优化，因为你可以向子组件传递 dispatch 而不是回调函数** 。


---
**useState版login**
我们先看看登录页常规的使用useState的实现方式：

```javascript
function LoginPage() {
        const [name, setName] = useState(''); // 用户名
        const [pwd, setPwd] = useState(''); // 密码
        const [isLoading, setIsLoading] = useState(false); // 是否展示loading，发送请求中
        const [error, setError] = useState(''); // 错误信息

        const login = (event) => {
            event.preventDefault();
            setError('');
            setIsLoading(true);
            login({ name, pwd })
                .then(() => {
                    setIsLoading(false);
                })
                .catch((error) => {
                    // 登录失败: 显示错误信息、清空输入框用户名、密码、清除loading标识
                    setError(error.message);
                    setName('');
                    setPwd('');
                    setIsLoading(false);
                });
        }
        return ( 
            //  返回页面JSX Element
        )
    }
```

**useReducer版login**
使用useReducer改造下上面的代码

```javascript
const initState = {
    name: '',
    pwd: '',
    isLoading: false,
    error: ''
}

function loginReducer(state, action) {
    switch(action.type) {
        case 'login':
            return {
                ...state,
                isLoading: true,
                error: '',
            }
        case 'success':
            return {
                ...state,
                isLoading: false,
            }
        case 'error':
            return {
                ...state,
                error: action.payload.error,
                name: '',
                pwd: '',
                isLoading: false,
            }
        default: 
            return state;
    }
}

function LoginPage() {
    const [state, dispatch] = useReducer(loginReducer, initState);
    const { name, pwd, isLoading, error, isLoggedIn } = state;
    const login = (event) => {
        event.preventDefault();
        dispatch({ type: 'login' });

        login({ name, pwd })
            .then(() => {
                dispatch({ type: 'success' });
            })
            .catch((error) => {
                dispatch({
                    type: 'error'
                    payload: { error: error.message }
                });
            });
    }
    return ( 
        //  返回页面JSX Element
    )
}
```

乍一看useReducer改造后的代码反而更长了，但很明显第二版有**更好的可读性**，我们也能更清晰的了解state的变化逻辑。login函数现在更清晰的表达了用户的意图，开始登录login、登录success、登录error。**LoginPage不需要关心如何处理这几种行为，那是loginReducer需要关心的，表现和业务分离**。

类似于状态模式，**每一种状态对应一组数据，状态变化，数据跟着变，对于1种状态对应多个数据变化时，reducer聚合了这种变化**，首先语义上更好，其次封装了颗粒度最小的useState（也就是单一数据）

---

**总结下使用reducer的场景**

 - 如果你的state是一个数组或者对象
 - 如果你的state变化很复杂，经常一个操作需要修改很多state
 - 如果你希望构建自动化测试用例来保证程序的稳定性
 - 如果你需要在深层子组件里面去修改一些状态（关于这点看链接）
 - 如果你用应用程序比较大，希望UI和业务能够分开维护

---

**useReducer + useContext的使用**

useReducer能帮助我们集中式的处理复杂的state管理。但如果我们的页面很复杂，拆分成了多层多个组件，我们如果在子组件触发这些state变化呢，比如在LoginButton触发登录操作？ **如何处理复杂组件树结构的reducer共享问题。**

```javascript
// 定义初始化值
const initState = {
    name: '',
    pwd: '',
    isLoading: false,
    error: ''
}
// 定义state[业务]处理逻辑 reducer函数
function loginReducer(state, action) {
    switch(action.type) {
        case 'login':
            return {
                ...state,
                error: '',
            }
        case 'success':
            return {
                ...state,
                isLoggedIn: true,
                isLoading: false,
            }
        case 'error':
            return {
                ...state,
                error: action.payload.error,
                name: '',
                pwd: '',
                isLoading: false,
            }
        default: 
            return state;
    }
}
// 定义 context函数
const LoginContext = React.createContext();
function LoginPage() {
    const [state, dispatch] = useReducer(loginReducer, initState);
    const { name, pwd, isLoading, error, isLoggedIn } = state;

    const login = (event) => {
        event.preventDefault();
        dispatch({ type: 'login' });

        login({ name, pwd })
            .then(() => {
                dispatch({ type: 'success' });
            })
            .catch((error) => {
                dispatch({
                    type: 'error'
                    payload: { error: error.message }
                });
            });
    }
    // 利用 context 共享dispatch
    return ( 
        <LoginContext.Provider value={dispatch}>
            // <...>
            <LoginButton />
        </LoginContext.Provider>
    )
}
function LoginButton() {
    // 子组件中直接通过context拿到dispatch，出发reducer操作state
    const dispatch = useContext(LoginContext);
    const click = () => {
        if (error) {
            // 子组件可以直接 dispatch action
            dispatch({
                type: 'error'
                payload: { error: error.message }
            });
        }
    }

    return <button onClick={click}>登录</button>
}

```

可以看到在useReducer结合useContext，通过context把dispatch函数提供给组件树中的所有组件使用 ，而不用通过props添加回调函数的方式一层层传递。

使用Context相比回调函数的优势：
1. 对比回调函数的自定义命名，Context的Api更加明确，我们可以更清晰的知道哪些组件使用了dispatch、应用中的数据流动和变化。这也是React一直以来单向数据流的优势。
2. 更好的性能：如果使用回调函数作为参数传递的话，因为每次render函数都会变化，也会导致子组件 re-render。当然我们可以使用useCallback解决这个问题，但相比useCallbackReact官方更推荐使用useReducer，因为React会保证dispatch始终是不变的，不会引起consumer组件的 re-render。

---

**useState和useReducer的区别**

useState可以看做是个**自带一个默认reducer函数的useReducer**，这个可以从源码看出来；