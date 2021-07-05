# useImperativeHandle

---

useImperativeHandle 可以让你在使用 ref 时自定义暴露给父组件的实例值。在大多数情况下，应当避免使用 ref 这样的命令式代码。useImperativeHandle 应当与 forwardRef 一起使用：

```
useImperativeHandle(ref, createHandle, [deps]);
```

**出现的原因：**在forwardRef例子中的代码实际上是不推荐的，因为无法控制要暴露给父组件的值，所以我们使用useImperativeHandle控制要将哪些东西暴露给父组件；

**带来的效果：**useImperativeHandle能控制要将哪些东西暴露给父组件，而不是盲目的暴露一个 DOM 或者 React组件实例；

[例子](https://codesandbox.io/s/useimperativehandle-u6ug7?file=/src/App.js)




