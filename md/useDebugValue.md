# useDebugValue

---

useDebugValue 可用于在 React 开发者工具中显示自定义 hook 的标签;
```javascript
useDebugValue(value)
```

**延迟格式化 debug 值**

useDebugValue 接受一个格式化函数作为可选的第二个参数，该函数只有在 Hook 被检查时才会被调用。它接受 debug 值作为参数，并且会返回一个格式化的显示值。

例如，一个返回 Date 值的自定义 Hook 可以通过格式化函数来避免不必要的 toDateString 函数调用：
```javascript
useDebugValue(date, date => date.toDateString());
```

例子稍后。。。