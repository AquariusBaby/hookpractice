# useCallback

----

接受一个 <u>内联回调函数</u> 和 <u>依赖项数组</u> 作为参数，返回该 <u>内联函数</u> 的memoized版本;

```javascript
const memoizedCallback = useCallback(
  () => {
    doSomething(a, b);
  },
  [a, b]
);
```

当你把回调函数传递给经过优化的并使用引用相等性去避免非必要渲染（例如 shouldComponentUpdate）的子组件时，它将非常有用。[看别人的例子](https://codesandbox.io/s/usecallback1-yu1sp)。

> * useCallback 本质是 useMemo 的语法糖，因为函数也是对象；
所以有：```useCallback(fn, deps)``` 相当于 ```useMemo(() => fn, deps)```
> * 注意```shallowEqual(() => {}, () => {}) // 浅比较下这两个函数是不相等的```；
> * useCallback 通常是搭配 React.memo 或者 shouldComponentUpdate 使用；

**Q1：是否需要把所有的方法都用 useCallback 包一层？**

A1: 不要把所有的方法都包上 useCallback

👇 这个情况就不没必要：
```
const [count1, setCount1] = useState(0);
const [count2, setCount2] = useState(0);

const handleClickButton1 = () => {
  setCount1(count1 + 1)
};
const handleClickButton2 = useCallback(() => {
  setCount2(count2 + 1)
}, [count2]);

return (
  <>
    <button onClick={handleClickButton1}>button1</button>
    <button onClick={handleClickButton2}>button2</button>
  </>
)
```

**Q2：每次函数组件重新渲染，里面的函数都会重建，是否需要用useCallback来减少重建？**

A1: 官方文档指出过，无需担心新建函数会导致性能问题，因为新建的消耗微乎其微

我们来看个例子：
```javascript
import React, { useCallback } from 'react';

function Comp() {
    const onClick = useCallback(() => {
        console.log('打印');
    }, []);
    
    return <div onClick={onClick}>Comp组件</div>
}
```

👆 这个用法，非但没性能提升，还不如不使用useCallback的情况；

我们来看下对上面的改写代码逻辑结构之后：

```javascript
import React, { useCallback } from 'react';

function Comp() {
    // 其实传给useCallback的内联回调函数，每次组件渲染都会重新声明，匿名不匿名函数都一样。useCallback只是对比依赖，判断是给你缓存的函数版本，还是新函数版本；
    const onClick = () => {
        console.log('打印');
    };
    
    const memoOnClick = useCallback(onClick, []); // 反倒多出一个函数执行
    
    return <div onClick={memoOnClick}>Comp组件</div>
}
```

**延伸下，useCallback源码**
// 待补充。。。


