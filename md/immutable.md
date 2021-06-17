#React中的不可变数据 - Immutable
- - -

与之相反的是：可变的 - variable

1. what is immutable?
2. why need immutable?
3. how use immutable?

----

###什么是不可变数据？

不可变数据的概念来源于**函数式编程**。在函数式编程中，对已初始化的**变量**是不可更改的，每次更改都要创建一个**新的变量**。

JavaScript在语言层面上并没有实现不可变数据，*怎么解决？*

1. 利用一些js提供的API。例如：数组中的slice、concat、ES6提供的扩展运算符等；
    - 排除掉7个操作数组且会改变原数组的方法：push、pop、unshift、shift、sort、splice、reserve
    - 当然这只能处理一些数组元素为基础类型的情况，遇到元素为引用类型的，这里面的数据要怎么处理？
2. 浅拷贝、深拷贝。没错，我们可以将一个复杂的引用类型数据完整的拷贝下来，再去修改这个副本，并返回即可；
    - 但深拷贝是有一定的损耗性能的，浅拷贝也只能适用于一层的嵌套情况；
    - 而且手写浅拷贝，要考虑的类型情况太多，也麻烦；
3. 借助第三方库<u>（如：immer.js、immutable.js）</u>；

---

###为什么React需要不可变数据？

使用不可变数据可以解决性能优化引入的问题，所以重点介绍这一部分背景。

<!-- 我们知道class组件和函数组件，每次都是默认render的，怎么理解？ -->
我们分别来看下 class组件 和 函数组件：

 1. class组件有个shouldComponentUpdate生命周期，其默认是返回true的，也就是说默认会调用class组件的render。
 而触发React组件render有三种方式：**props改变**、**setState**、**父组件的render函数执行**。
    - **props改变**：也就是父组件传递的props变了（基础类型的值变了，引用类型的引用地址变了），因为shouldComponentUpdate默认返回true，所以组件会重新执行render函数；
    - **setState**：这个自己主动触发的，到没什么可讲的；
    - **父组件的render函数执行**：即使子组件上没有任何父组件传递的props，但父组件的render函数执行，会解析到子组件，子组件也会执行，也会走生命周期，还是因为shouldComponentUpdate默认返回true，所以子组件会重新执行render函数。*（ 当子组件什么都不传的时候nextProps={},this.props={},两个{}都是新创建的{}!=={} ）*   

    **所以，为了解决这种不必要的渲染，有两种方式：**
    1. shouldComponentUpdate中自行加入判断和检测；
    2. React15.3版本引入的PureComponent。原理和1一样，不过是官方提供的判断和检测方式，采用的是**浅比较**（注意：浅比较的是this.state这一层哦）；实现原理请看**shallowEqual.md文档**；
 2. 函数组件，没有生命周期的概念，也更没有shouldComponentUpdate一说，所以函数组件通常被用作展示组件，不涉及状态管理。但同样，既是每次传递的props相同，依旧会重新执行函数组件本身，并返回JSX对象。
    - 为了解决这种情况，React引入React.memo(component, areEqual)接受一个函数组件，默认采用浅比较，当然第二个参数areEqual可以自定义比较和判断。
    - 因为函数组件没有state，所以React.memo只比较props。
    - 还有就是，如果 props 相等，areEqual 会返回 true；如果 props 不相等，则返回 false。这与 shouldComponentUpdate 方法的返回值相反。
    - React.memo 为高阶组件。（接受一个组件，返回一个组件；源码上走bailoutOnAlreadyFinishedWork）；
    - *推荐使用 React.useMemo 而不是 React.memo，因为在组件通信时存在 React.useContext 的用法，这种用法会使所有用到的组件重渲染，只有 React.useMemo 能处理这种场景的按需渲染。（有待考量）*
        ```
        // 像这样
        const App: React.FC<{ title: string }> = ({ title }) => {
            return React.useMemo(() => <div>{title}</div>, [title]);
        };

        App.defaultProps = {
            title: 'Function Component'
        }
        ```
    - 因为函数组件被 React.memo 包裹，且其实现中拥有 useState，useReducer 或 useContext 的 Hook，当 context 发生变化时，它仍会重新渲染。因为React.memo只判断props，不判断context；
    - 如何处理，可以参考[这个](https://github.com/facebook/react/issues/15156)

---- 

这样是没问题的，挺好的，但我们来看看这种性能优化引入的问题：
```
class Child extends React.PureComponent {
  render() {
    return <div>{this.props.people.name}</div>;
  }
}

class Parent extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      people: {name: '杰克'}
    };
    this.handleClick = this.handleClick.bind(this);
  }

  handleClick() {
    // 这部分代码很糟，而且还有 bug
    const people = this.state.people;
    people.name = '瑞克';

    this.setState({words: words});
  }

  render() {
    return (
      <div>
        <button onClick={this.handleClick} />
        <Child words={this.state.words} />
      </div>
    );
  }
}
```
当我们这里采用可变数据修改的方式去修改数据的时候，this.state.people的引用并没有发生变化。因此会在PureComponent的shouldComponentUpdate阶段返回false，这就导致视图渲染的还是"杰克"，视图渲染就不正确了；

所以解决方案就是使用不可变数据；

通常的情况，使用ES6的解构，数组的slice之类的方法；
但最透彻的方案就是使用第三方库<u>（如：immer.js、immutable.js）</u>；

**immutable.js**

 - 自己维护了一套数据结构，Javascript 的数据类型和 immutable.js 的类型需要相互转换，对数据有侵入性。
 - 库的体积比较大（63KB），不太适合包体积紧张的移动端。
 - API 极其丰富，学习成本较高。
 - 兼容性非常好，支持 IE 较老的版本。

**immer.js**

 - 使用 Proxy 实现，兼容性差。
 - 体积很小（12KB），移动端友好。
 - API 简洁，使用 Javascript 自己的数据类型，几乎没有理解成本。

优缺点对比之下，immer 的兼容性缺点在我们的环境下完全可以忽略。使用一个不带来其他概念负担的库还是要轻松很多的。