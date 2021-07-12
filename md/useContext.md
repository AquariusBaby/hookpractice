# useContext

----

> 调用了 useContext 的组件总会在 context 值变化时重新渲染，[可以看例子](https://codesandbox.io/s/usecontextyureactmemo-3tgoz?file=/src/App.js)。如果重渲染组件的开销较大，你可以 通过使用 memoization 来优化。<u>这个优化的例子可以看</u>[dan总的3个option](https://github.com/facebook/react/issues/15156)

context的定义：
```javascript
// context.js
import React from 'react';

// 只有当组件所处的树中没有匹配到 Provider 时，其 defaultValue 参数才会生效；
// 换句话说，如果你在当前组件去订阅了Context对象，但你当前的组件并未包裹在Provider下，
// 访问到的就是defaultValue值了。（有助于在不使用 Provider 包装组件的情况下对组件进行测试）
const default = {....};
const MyContext = React.createContext(default);
```

 - class组件：Component.contextType = MyContext; 或者 static contextType = MyContext;
 - 函数组件：
    ```
    <MyContext.Consumer>
        {(context) => <div>{context.name}</div>}
    </MyContext.Consumer>
    ```
    或者使用useContext;

[对应的context的4种写法](https://codesandbox.io/s/contextheusecontext-o05ol?file=/src/App.js)


> * 当 Provider 的 value 值发生变化时（ 这里的对比方式是<u>Object.is(obj, obj)</u> ），它内部的所有消费组件都会重新渲染。// (源码上有时间可以看看)
> * Provider 及其内部 consumer 组件都不受制于 **shouldComponentUpdate 函数**、也不受制于 **React.memo 的第二个比较函数**。
> * 因此当 consumer 组件在其祖先组件退出更新的情况下也能更新 **（这里的意思是，哪怕consumer组件的父级的 shouldComponentUpdate 返回false 或者 React.memo 的第二个参数返回true，也照样更新）** （这是新context，但旧的context与之相反，所以之前一直不被官方推荐使用）[新老context的对比](https://juejin.cn/post/6907546624441090055)。
> * 所以要注意传递对象的情况

```javascript
<!-- 所以你应该避免这么做： -->
<MyContext.Provider value={{something: 'something'}}>
    <Toolbar />
</MyContext.Provider>
```

// 待补充：react-redux的实现原理
<!-- 这个时候你可能会想到react-redux是怎么做的： -->

<!-- [新老context的API对比]https://juejin.cn/post/6844903683931504647 -->


