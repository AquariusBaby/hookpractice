**什么是reducer**

reducer的概念是伴随着Redux的出现逐渐在JavaScript中流行起来。但我们并不需要学习Redux去了解Reducer。简单来说 reducer是一个函数(state, action) => newState：接收当前应用的state和触发的动作action，计算并返回最新的state。

**reducer 的幂等性**

从上面的示例可以看到reducer本质是一个纯函数，没有任何UI和副作用。这意味着相同的输入（state、action），reducer函数无论执行多少遍始终会返回相同的输出（newState）。因此通过reducer函数很容易推测state的变化，并且也更加容易单元测试。

```
expect(countReducer(1, { type: 'add' })).equal(2); // 成功
expect(countReducer(1, { type: 'add' })).equal(2); // 成功
expect(countReducer(1, { type: 'sub' })).equal(0); // 成功
```

**reducer 是纯函数**
1. reducer处理的state对象必须是immutable，这意味着永远不要直接修改参数中的state对象，reducer函数应该每次都返回一个新的state object；
2. 对于这种复杂state的场景推荐使用immer等immutable库解决。

**state为什么需要immutable？**
1. reducer的幂等性
reducer需要保持幂等性，更加可预测、可测试。如果每次返回同一个state，就无法保证无论执行多少次都是相同的结果

2. React中的state比较方案
React在比较oldState和newState的时候是使用Object.is函数，如果是同一个对象则不会触发组件的re-render。 可以[参考官方文档bailing-out-of-a-dispatch](https://reactjs.org/docs/hooks-reference.html#bailing-out-of-a-dispatch)。

**action 的理解**
action：用来表示触发的行为。
1. 用type来表示具体的行为类型(登录、登出、添加用户、删除用户等)
2. 用payload携带的数据（如增加书籍，可以携带具体的book信息）


简单总结一下：reducer是一个利用action提供的信息，将state从A转换到B的一个纯函数，具有一下几个特点：
 - 语法：(state, action) => newState
 - Immutable：每次都返回一个newState， 永远不要直接修改state对象
 - Action：一个常规的Action对象通常有type和payload（可选）组成
    - type：本次操作的类型，也是 reducer 条件判断的依据
    - payload：提供操作附带的数据信息