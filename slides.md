---
# try also 'default' to start simple
theme: Seriph
# random image from a curated Unsplash collection by Anthony
# like them? see https://unsplash.com/collections/94734566/slidev
background: https://source.unsplash.com/collection/94734566/1920x1080
# apply any windi css classes to the current slide
class: "text-center"
# https://sli.dev/custom/highlighters.html
highlighter: shiki
# show line numbers in code blocks
lineNumbers: true
# some information about the slides, markdown enabled
info: |
  ## React Hooks 的使用分享
---

# React Hooks 分享

数据治理产品部 夫间 1835

<div class="pt-12">
  <span @click="$slidev.nav.next" class="px-2 py-1 rounded cursor-pointer" hover="bg-white bg-opacity-10">
    开始 <carbon:arrow-right class="inline"/>
  </span>
</div>

---

# React Hooks 是干啥的？

Hook 是 React 16.8 的新增特性。它可以让你在不编写 class 的情况下使用 state 以及其他的 React 特性。

## 优点

<br />

1. 可重用性
2.  可读性
3.  可测试性

<br />
<br />


<v-click>

## 少写代码😁 -> 少加班😎

</v-click>


<!--
You can have `style` tag in markdown to override the style for the current page.
Learn more: https://sli.dev/guide/syntax#embedded-styles
-->

<style>
h1 {
  background-color: #2B90B6;
  background-image: linear-gradient(45deg, #4EC5D4 10%, #146b8c 20%);
  background-size: 100%;
  -webkit-background-clip: text;
  -moz-background-clip: text;
  -webkit-text-fill-color: transparent;
  -moz-text-fill-color: transparent;
}
</style>

---

# 导览

1. **useState**

2. **useEffect & useLayoutEffect**

3. **useRef & useImperativeHandle**

4. **useMemo & useCallback**

5. **useReducer**

6. **useContext**

7. **自定义Hooks**

---

# 注意事项

* 只在函数组件中调用 Hook
* 不能在循环、条件或嵌套函数中调用 Hooks  <!-- 因为hooks 总是按照顺序被调用 -->
* 搭配 eslint 插件使用

<br/>

```shell
npm install eslint-plugin-react-hooks --save-dev
```

```json
// 你的 ESLint 配置
{
  "plugins": [
    // ...
    "react-hooks"
  ],
  "rules": {
    // ...
    "react-hooks/rules-of-hooks": "error", // 检查 Hook 的规则
    "react-hooks/exhaustive-deps": "warn" // 检查 effect 的依赖
  }
}
```

---

# useState

为函数组件添加状态

```js {monaco}
import { useState } from "react";
function Counter() {
  // 这里可以任意命名，比如updateCount，因为返回的是数组，数组解构
  const [count, setCount] = React.useState(0)
  const increment = () => setCount(count + 1)
  return <button onClick={increment}>{count}</button>
}
```

参数：`initialState` 可以是任何值

返回值：`[state, state_dispatch_function]` (数组)

---

# useState 使用注意点

## 1. 当属性为 Object 时，使用不可变数据 immutable data

React 使用 `Object.is` 来比较数据

```js {monaco}
const Message = () => {
  const [messageObj, setMessage] = useState({ message: '' });
  return (
    <div>
      <input
        type="text"
        value={messageObj.message}
        onChange={e => {
          // 无法触发更新
          messageObj.message = e.target.value;
          setMessage(messageObj);
        }}
      />
  </div>
  );
};
```

---

## 2. `useState` 是异步更新的

React 的 batch update 机制

```js
function Counter() {
  const [count, setCount] = React.useState(0)
  const increment = () => {
    setCount(count + 1)
    // 获取不到最新的 count
    console.log(count)
  }
  return <button onClick={increment}>{count}</button>
}
```

<br />

1. 使用 函数式更新 `prevState => preSate + 1`
2. 使用 useRef

[举个例子 🙋‍♂️🌰](https://codesandbox.io/s/aged-resonance-rdjfs?file=/src/App.js)

---

## 3. Lazy initialization useState 的初始化

```js
// 执行一个 IO 操作
const [count, setCount] = React.useState(Number(window.localStorage.getItem('count')))

const getInitialState = () => Number(window.localStorage.getItem('count'))
const [count, setCount] = React.useState(getInitialState)

```

性能优化的一种方式

[举个例子 🙋‍♂️🌰](https://codesandbox.io/s/eager-torvalds-z0egg?file=/src/App.js)

---

# useEffect & useLayoutEffect

处理副作用的函数，纯函数中和入参无关的处理值（数据获取，dom 修改）

```js
function Counter() {
  const [count, setCount] = useState(0);

  useEffect(() => {
    document.title = `You clicked ${count} times`;
  });

  return (
    <div>
      <p>You clicked {count} times</p>
      <button onClick={() => setCount(count + 1)}>
        Click me
      </button>
    </div>
  );
}
```

---

# 注意点

1. 小心无限循环
2. 注意调用的顺序
3. 不正确的使用 async await

codesandbox

---

# useLayoutEffect

99% 的情况下使用 useEffect, 两者的 api 相同

差异
useEffect 是异步执行的，而useLayoutEffect是同步执行的。

<div class="flex mt-8">

<div v-click class="flex-grow w-1/2 pr-2">

useEffect:

1. 触发渲染
2. React 把 vdom -> dom
3. 屏幕更新
4. useEffect 运行

</div>

<div v-click class="flex-grow w-1/2 border-l border-vgreen pl-2">

useLayoutEffect:

1. 触发渲染
2. React 把 vdom -> dom
3. useLayoutEffect 运行
4. 屏幕更新

</div>

</div>

例子：演示demo https://www.youtube.com/watch?v=R6zvdn40VfQ

---

# useRef & useImperativeHandle

```js
const refContainer = useRef(initialValue);
// refContainer: { current: initialValue }
```

* 可以保存任何值
* 与 { current: ''} 对象的区别？
* 不会触发组件的重新渲染 (尽量不要在 UI 中使用，最好把改动动作放到 useState 前)

### 使用的地方

1. 在 Hooks 中作为一个全局变量使用。
2. 控制 dom 节点 <!-- 小写的 jsx 标签 -->，控制子组件(配合 useImperativeHandle)

---

# useImperativeHandle

```js
useImperativeHandle(ref,create,[deps])

// 第1个参数为父组件通过useRef定义的引用变量；
// 第2个参数为子组件要附加给ref的对象，该对象中的属性即子组件想要暴露给父组件的函数(方法)；
// 第3个参数为可选参数，为函数的依赖变量。凡是函数中使用到的数据变量都需要放入deps中，如果处理函数没有任何依赖变量，可以忽略第3个参数。
```

函数式组件是没有实例的，所以我们不能直接通过 ref 来调用子组件的方法。useImperativeHandle可以让父组件获取并执行子组件内某些自定义函数(方法)。本质上其实是子组件将自己内部的函数(方法)通过useImperativeHandle添加到父组件中useRef定义的对象中。

## 使用流程

1. useRef 创建引用变量
2. React.forwardRef 将引用变量传递给子组件
3. useImperativeHandle将子组件内定义的函数作为属性，添加到父组件中的ref对象上。


[CodeSandBox] 查看例子

---

# useMemo & useCallback

第二个例子:

```js
import React from "react";
export default function App() {
  const [count, setCount] = React.useState(0);
  const value = { name: 1 };  // 每次渲染都是重新声明, Object.is 的比较

  React.useEffect(() => {
    setCount(Math.random());
    ("render");
  }, [value]);
  return (
    <div className="App">
      <h1>Hello CodeSandbox</h1>
      <h2>Edit to see some magic happen!</h2>
    </div>
  );
}
```

<v-click>

App渲染几次，value被定义几次， alert 会被弹出几次?

</v-click>

<v-click>

无限循环，全都无限次, 组件渲染 → useEffect执行 → setCount触发循环 → 组件渲染 → useEffect执行 → setCount触发循环...

</v-click>

---

解决方法:

```js
import "./styles.css";
import React from "react";

export default function App() {
  const [count, setCount] = React.useState(0);
  const value = React.useMemo(() => {
    return { name: 1 };
  }, []);

  React.useEffect(() => {
    setCount(Math.random());
    alert("render");
  }, [value]);
  return (
    <div className="App">
      <h1>Hello CodeSandbox</h1>
      <h2>Edit to see some magic happen!</h2>
    </div>
  );
}
```

useMemo 的意思就是：不要每次渲染都重新定义，而是我让你重新定义的时候再重新定义(第二个参数，依赖列表)。大家看到这里的依赖列表是空的，是因为useMemo里的回调函数确实没用到啥变量，如果有变量的话大家的IDE就会提醒加上依赖了。

这就是使用useMemo的原理，useMemo适用于所有类型的值，加入这个值恰好是函数，那么用useCallback也可以。也就是说，useCallback是一种特殊的useMemo。

在这里再粗暴地给大家总结一下日常使用的场景：

如果你定义了一个变量，满足下面的条件就最好用useMemo和useCallback给包裹住：

1. 它不是状态，也就是说，不是用useState定义的(redux中的状态实际上也是用useState定义的)
2. 它不是基本类型
3. 它会被放在useEffect的依赖列表里 || 自定义hook的返回值

---

使用注意点:

1. 不需要在所有函数中使用

---

# useContext

解决组件之间的共享状态

```js
const MyContext = React.createContext('')
const value = useContext(MyContext);

<UserContext.Provider value={'chuanshi'}>
  <ComponentC />
</UserContext.Provider>
```

找个例子

---

# useReducer

useReducer是React提供的一个高级Hook

https://juejin.cn/post/6844903869604986888

---

# 自定义 Hooks

通过自定义 Hook，可以将组件逻辑提取到可重用的函数中。

例子

```js
  const [form] = Form.useForm(); // dtd 2.0 中的表格交互
```

实现一个 useTitle 的自定义 hooks
https://ahooks.js.org/zh-CN/hooks/dom/use-title

---

# 更多

---

# 作业
