# shallowEqual浅比较函数
---

 - 浅比较：浅比较就是只比较第一级;
    1. 对于基本数据类型，只比较值；
    2. 对于引用数据类型值，直接比较地址是否相同，不管里面内容变不变，只要地址一样，我们就认为没变；
 - Object.is(obj, obj)：用来比较两个值是否严格相等，与严格比较运算符（===）的行为基本一致；不过稍微有点不同；
    1. 一是+0不等于-0；
    2. 二是NaN等于自身；

--- 

#### 先实现下 Object.is(obj, obj):
> * 严格比较运算符（===）
+0 === -0 // true
+0 === 0 // true
NaN === NaN // false

> * ES6的 Object.is()
Object.is(+0, -0) // false
Object.is(NaN, NaN) // true

我们用ES5来实现ES6的这个方法

```
function is(x, y) {
    if (x === y) {
        // 针对+0 不等于 -0的情况；
        // 1/-0 -Infinity 负无穷
        // 1/+0 Infinity 正无穷
        return x !== 0 || y !== 0 || 1/x === 1/y;
    } else {
        // 针对NaN的情况
        return x !== x && y !== y;
    }
}
```

#### 再来实现 shallowEqual：

```
function shallowEqual(objA, objB) {
    // 如果两者都是基础类型且相等，或者指向同一个对象
    if (is(objA, objB)) return true;

    // 如果objA, objB有一个是基础类型，另一个是对象，那么返回false
    if (
        typeof objA !== "object" ||
        objA === null ||
        typeof objB !== "object" ||
        objB === null
    ) {
        return false;
    }

    if (keysA.length !== keysB.length) return false;

    for (let i = 0; i < keysA.length; i++) {
        if (
            !Object.prototype.hasOwnProperty.call(objB, keysA[i]) ||
            !is(objA[keysA[i]], objB[keysA[i]])
        ) {
            return false;
        }
    }

    return true;
}
```
