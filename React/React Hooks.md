# React Hooks

#### Hooks 要解决的问题：

跨组件地复用包含状态的逻辑，通过 Hooks 可以将含有 state 的逻辑从组建抽象出来，同时也可以帮助我们在*不重写组件结构*的情况下复用逻辑。Hooks 一般是用于函数式组件的，在类class组件中无效。让我们根据代码的作用将它们拆分，而不是生命周期。

#### Hooks 的语法规则

- 只能在顶层调用钩子。不在循环、控制流和嵌套的函数中调用钩子。
- 只能从React的函数式组件中调用钩子。不在常规的JS函数中调用钩子。

#### 创建Hooks

- 使用`useState`创建Hook

```react
import {useState} from 'react';

function hooks(){
    // 声明一个名为 count 的新状态变量
    const [count, setCount] = useState(0);
    // 第二个参数 setCount 为一个可以更新状态的函数
    // useState 的参数即为初始值
    
    return (
        <div>
        	<p>当前的状态量为: {count}</p>
            <button onClick={() => setCount(count + 1)}>点击加一</button>
        </div>
    )
}
```

- 使用 `useEffect` 来执行相应操作

```react
import {useState, useEffect} from 'react';

function hooks(){
    const [count, setCount] = useState(0);
    // 类似于 componentDidMount 和 componentDidUpdate
    // 在 useEffect 中可以使用组建的 state 和 props
    // 在每次渲染后都执行 useEffect
    useEffect(() => {
        window.alert(`You have clicked ${count} times`);
    })
    return (
        <div>
        	<p>当前的状态量为: {count}</p>
            <button onClick={() => setCount(count + 1)}>点击加一</button>
        </div>
    )
}
```

#### 钩子是独立的

我们在两个不同的组件使用同一个钩子，他们是相互独立的，甚至在一个组件使用两个钩子他们也是相互独立的，

##### React如何保证useState相互独立

React 其实是根据`useState`传出现的顺序来保证`useState`之间相互独立。

```react
// 首次渲染
const [num, setNum] = useState(1); // 将num初始化为1
const [str, setStr] = useState('string'); // 将str初始化为'string'
const [obj, setObj] = useState({id:1}); // ....
// 第二次渲染
const [num, setNum] = useState(1); // 读取状态变量num的值， 此时传入的参数已被忽略，下同
const [str, setStr] = useState('string'); // 读取状态变量str的值
const [obj, setObj] = useState({id:1}); // ....
```

同时正是由于根据顺序保证独立，所以 React 规定我们必须把 hooks 写在最外层，而不能写在条件语句之中，来确保hooks的执行顺序一致，若要进行条件判断，我们应该在 `useEffect` 的函数中写入条件

#### Effect Hooks

useEffect 来传递给 React 一个方法，React会在进行了 DOM 更新之后调用。我们通常将 useEffect 放入组件内部，这样我们可以直接访问 state 与 props。记得，useEffect 在每次 render 后都要调用。

##### 需要清理的Effect

我们有时需要从*外部数据源*获取数据，此时我们就要保证清理Effect来避免内存泄露 ，此时我们需要在 effect 中返回一个函数来清理它， React 会在组件每次接触挂载的时候清理。一个比较使用的场景就是我们在 `useEffect`中若执行了异步请求，由于异步的时间不确定性，我们很需要在执行下一次异步请求时先结束上一次的请求，因此我们就需要清理。

```react
useEffect(() => {
    let canceled = false;
    const getData = async () => {
        const res = await fetch(api);
        if(!canceled) {
            // 展示 res
        }
    }
    
    getData();
    
    // return 的即为我们的清理函数
    return () => {
        canceled = true;
    }
});
```

此时我们在进行重新渲染时，就可以避免异步请求带来的竞态问题，从而避免数据的不稳定性。

##### 配置根据条件执行的Effect

我们可以给`useEffect`传入第二个参数只有当第二个参数（数组）里的所有的state 值发生变化时，才重新执行Effect

```react
useEffect(() => {
    window.alert(`you had clicked ${count} times`);
}, [count]); //只有当 count 发生变化时才会重新执行effect
```



### Hooks 的好处

- 避免了我们在复用含状态组件(classes) 时使用 `render props` 与 `高阶组件`时产生的

夸张的层级嵌套。

- 防止我们为了实现功能而在生命周期函数中写入了大量的重复的代码。
- classes 中的 this 指向十分的迷惑。



