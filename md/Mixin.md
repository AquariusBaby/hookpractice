# Mixins

Mixin（混入）是一种通过扩展收集功能的方式，它本质上是将一个对象的属性拷贝到另一个对象上面去，不过你可以拷贝任意多个对象的任意个方法到一个新对象上去，这是继承所不能实现的。它的出现主要就是为了解决代码复用问题。

---

### 原理：
```javascript
function Mixin(target, mixin) {
  for (let prop in mixin) {
    if (Object.hasOwnProperty(mixin, prop)) {
      target.prototype[prop] = mixin[prop];
    }
  }

  return target;
}
```
----

### 缺陷：
- **Mixin 可能会相互依赖，相互耦合，不利于代码维护;**
- **不同的Mixin中的方法可能会相互冲突;**
    假如你在Mixin1中定义了一个 getDefaultProps，另外一个Mixin2又定义了同样的名称getDefaultProps, 造成了冲突。
- **Mixin非常多时，组件是可以感知到的，甚至还要为其做相关处理，这样会给代码造成滚雪球式的复杂性;**