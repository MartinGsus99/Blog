# React Hooks

https://zhuanlan.zhihu.com/p/429308983

## 背景

React-Hooks 是 React 团队在组件开发实践中，逐渐认知到的一个改进点，这背后其实涉及对**类组件**和**函数组件**两种组件形式的思考和侧重

## 类组件

```jsx
class Test extends React.Component {
  constructor(props) {
    super(props)
    this.state = { count: 0 }
    this.change = this.change.bind(this)
  }

  componentDidMount() {
    console.log('mount');
  }

  change(){
    this.setState({count: this.state.count+1})
  }

  render() {
    return <div>
      {this.state.count}
      <button onClick={this.change}>click</button>
    </div>
  }
}
```



## 函数组件

> 就是以函数形态存在的组件，因为一开始并没有 hook，所以函数组件无法定义和维护state，被称为无状态组件

```jsx
function Test(props) {
  const { value } = props
  return <div className="wrapper">
    <span>get some value: {value}</span>
  </div>
}
```

在 hook 出现之前，类组件要明显强于函数组件，函数组件最大的问题是无法维护内部状态

react hooks 的出现，可以让我们在不编写 class 的情况下使用 state 以及其他的 React 特性，补齐函数相对于类组件而言缺失的功能。

没有太多书写的限制，不强制按照生命周期划分逻辑，不需要理解 this，将复杂组件中相互关联的部分拆分成更小的函数达到复用的目的。



## Vue怎么做的？

类组件和函数组件之间，是面向对象和函数式编程这两套不同的设计思想之间的差异

- react 16.8 新增 hook 大力推进 函数组件的使用
- vue3 新增 composition API 取代 options API 的写法

![img](https://pic1.zhimg.com/80/v2-67c2f64139aff452373da41bc93d84a4_720w.webp)

### options API对比React类组件

1. 组件数据：vue data类似于this.state,数据必须在这里初始化
2. 功能方法：vue限制在methods，react将所有方法分散在class里
3. 生命周期：vue和react都是通过特定方法名调用

#### 存在的问题

1. 逻辑不好拆分达到复用，和组件强关联
2. 不相关的代码组合在一起，相关的代码反而聚合在一i；
3. 需要注意this指向；

#### 解决方法

- vue：引入mixin、extends等，增加了使用和维护成本，带来了数据流向不清晰的问题；
- react：引入providers，consumers，告诫组件，render props等抽象层组成的组件，形成嵌套地狱，也存在数据流向的问题；

#### 结果

- vue3引入了composition API与Setup
- react引入hook

## Hooks

### 分类

- 组件状态处理相关：useState、useReducer、useContext
- 处理副作用：useEffect、useLayoutEffect
- 性能优化相关：useMemo、useCallback
- DOM相关：useRef
- redux相关：useSelector、useDispatch、useStore
- 用户自定义hook等

#### useState

> 在函数组件保存数据的主要方法，等同于类组件的 this.state 与 this.setState

```react
  const [count, setCount] = React.useState(0)
```

接受初始值，返回一个state以及更新函数；初始值可以为一个函数，但是其返回需要时数组；



#### useEffect

> 类组件中在生命周期中执行副作用，useEffect的作用是补充函数组件无法正确执行副作用的问题；
>
> 在 React 组件中执行过数据获取、订阅或者手动修改过 DOM。我们统一把这些操作称为“副作用”，或者简称为“作用”。

```react
  const fetchData = () => {
    console.log("fetchData")
  }

  useEffect(() => {
    fetchData()
  }, [])

```

useEffect接受两个参数：

- 被监听的参数发生变化时执行回调函数；
- 被监听的参数

当监听参数发生变化时就会执行回调，这里的空数组只会在初次渲染时执行，等同于`componentDidMount`

useEffect善用参数1、2可以代替大部分生命周期；

1.  componentDidMount 如上面实例，参数为空数组，表示不依赖任何数据，只在初次渲染后触发
2.  同时代替 componentDIdMount 和 componentDidUpdate， 有三种场景

```react
// 没有指定state
React.useEffect(() => {
  console.log('任意state发生变化都会触发, 包括初始化,  componentDIdMount + componentDidUpdate')
})

// 指定 state
React.useEffect(() => {
  console.log("只有当n发生变化才会触发，包括初始化，componentDIdMount + componentDidUpdate")
}, [n])

React.useEffect(() => {
  console.log("只有当 n 或 x 发生变化才会触发，包括初始化，componentDIdMount + componentDidUpdate")
}, [n, x])

// 指定state，回调有返回新的函数
React.useEffect(() => {
  function change n() {}
  SomeAction.subscribe(change, n) // 重新订阅 n
  return () => {
     SomeAction.unSubscribe(change, n) // 取消订阅 n
  }
}, [n]) // 假设 n 和监听器或者定时器等有关联，n 变化后需要重新订阅，或者是重启定时器之类的情况
```

3. 代替componentUllUnmount

```react
React.useEffect(() => {
  console.log('空数组，仅第一次执行，componentDIdMount')
  const onResize = (e) => {}
  window.addEventListener('resize', onResize)
  return ()=>{
    window.removeEventListener('resize', onResize)
    console.log('会在组件卸载调用, componentWillUnmount');
  }
}, [])
```

#### useMemo

> 解决函数组件的性能问题，比如子组件重复执行问题，每次渲染都进行高开销的计算

```react
function Sub(props) {
 console.log("Sub render");
  let { number, onClick } = props
  return (
    <button onClick={onClick}>{number}</button>
  )
}

function Test() {
  let [value, setValue] = React.useState('')
  let [number, setNumber] = React.useState(0)
  const addClick = () => setNumber(number + 1)
  return <>
    <input
      type="text"
      value={value}
      onChange={(e) => setValue(e.target.value)}
    />
    <Sub number={number} onClick={addClick} />
  </>
}
```

上面代码，子组件依赖只有number，理想情况下只希望number变化时触发子组件重新渲染，但是实际是在输入框内的值发生变化，子组件也会冲洗你渲染，class组件使用`shoouldComponentUopdate(nextProp,nxtState)`生命周期，在组件更新之前，判断当前组件是否受某个state或prop更改的影响。

> 在函数组件中，不区分mount和update两个状态，这意味着函数组件的每一次调用都会执行内部所有逻辑，带来巨大的性能损耗；
>
> useMemo和useCallback都是解决上述性能问题的

#### React.memo

```jsx
import { useState } from 'react'
import { memo } from 'react'
const Child = memo((props) => {
  console.log('Child render')

  return (
    <>
      <div>子组件</div>
    </>
  )
})

export default Child

```

```jsx
  const [p1, setP1] = useState("Hello")
  const [p2, setP2] = useState("World")
  //p4每次在父组件重新渲染的时被重新声明，导致子组件认为更新了；
  const p4 = () => {
    console.log("p1", p1, "p2", p2)
  }
  return (
    <div className="App">
      <div className="mainView">
        <div>我是父组件，count: {count}, p1: {p1}, p2: {p2}</div>
        <MyButton type="primary" text="点击+1" clickFunction={() => setCount(count + 1)} />
        <Child p1={p1} p2={p2} p4={p4} />

      </div>
    </div>
  )

```

使用useCallbck

```jsx
  const [p1, setP1] = useState("Hello")
  const [p2, setP2] = useState("World")

  const p4 = useCallback(() => {
    console.log("p1", p1, "p2", p2)
  }, [p1, p2])
  return (
    <div className="App">
      <div className="mainView">
        <div>我是父组件，count: {count}, p1: {p1}, p2: {p2}</div>
        <MyButton type="primary" text="点击+1" clickFunction={() => setCount(count + 1)} />
        <Child p1={p1} p2={p2} p4={p4} />

      </div>
    </div>
  )
}
```



#### useCallback

> useCallback(fn,deps)相当于useMemo(()=>fn,deps)
>
> useMemo返回一个值，可能是表示数组的对象，useCallback返回一个函数。

所有依赖本地状态或props来创建函数，需要使用到缓存函数的地方，都是useCallback的应用场景

#### useRef

> 类组件中createRef生成DOM节点的引用，在函数组件中也可以使用；

```react
  let [text, setText] = useState('')
  const textRef = useRef()

  useEffect(() => {
    textRef.current = text
    console.log(textRef)
  }, [text])


  <input ref={textRef} value={text} onChange={e => setText(e.target.value)}></input>
        <p>当前内容：{text}</p>
```

#### useContext

> 优化了函数组件context的能力，并进行了写法上的统一；
>
> context 类似于 vue provide的概念，无需将组件传递到每一个组件，跨组件级传递变量，实现共享

#### useReducer

> 类似于状态机，有不同的状态，并且有修改状态的方法；

```js
function countReducer(state, action) {
    switch(action.type) {
        case 'add':
            return state + 1;
        case 'sub':
            return state - 1;
        default: 
            return state;
    }
}
```

useReducer可以增强函数组件中reducer的使用

```react

import './App.css'
import MyButton from './components/Button'
import Modal from './components/Modal'
import { useState, useMemo, useEffect, useRef, useReducer } from 'react'

import { useSelector, useDispatch } from 'react-redux'
import { increment, decrement } from './store/counter'

function App () {
  const [count, dispatch] = useReducer((state, action) => {
    switch (action) {
      case 'increment':
        return state + 1
      case 'decrement':
        return state - 1
      default:
        return state
    }
  }, 0)


  return (
    <div className="App">
      <div className="mainView">
        <p>{count}</p>
        <MyButton clickFunction={() => dispatch('increment')} text={'Increment'} />
        <MyButton clickFunction={() => dispatch('decrement')} text={'Decrement'} />
      </div>


    </div>
  )
}

export default App

```







