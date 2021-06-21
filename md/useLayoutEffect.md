# useLayoutEffect

其函数签名与 useEffect 相同，但它会在所有的 DOM 变更之后同步调用 effect。可以使用它来读取 DOM 布局并同步触发重渲染。在浏览器执行绘制之前，useLayoutEffect 内部的更新计划将被同步刷新。

和useEffect很相似，但执行时机不同，前者被异步调度，当页面渲染完成后再去执行，不会阻塞页面渲染。 后者是在commit阶段新的DOM准备完成，但还未渲染到屏幕之前，同步执行（所以useLayoutEffect 会阻塞渲染，应该避免使用一些耗时的操作）。

所以useLayoutEffect你可以看成是requestAnimationFrame的实现；

[运用场景一般是动画](https://codesandbox.io/s/uselayouteffect-7c40g?file=/src/App.js)