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

## 优点 ⚛️ + 🪝

<br />

1. 可重用性
2. 可读性
3. 可测试性
4. 不用再考虑 this 的问题啦

<br />
<br />

<v-click>

## 少写代码 😁 -> 少加班 😎

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

1. **useState** 🤩

2. **useEffect & useLayoutEffect** 🤩

3. **useRef & useImperativeHandle** 🤩

4. **useMemo & useCallback** 🤩

<!-- 5. **useReducer**

6. **useContext** -->

5. **自定义 Hooks** 🤩

---

# 注意事项

- 只在函数组件中调用 Hook
- 只能在函数组件的最顶层使用 hooks，而不能再 for 循环、if 等语句下面使用 hooks [🌰](https://www.yuque.com/erzhuyijian/mb596i/uduqea)<!-- 因为hooks 总是按照顺序被调用 -->
- 搭配 eslint 插件使用

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

# 在开始之前，理清概念

来看看渲染

```js {all|6}
function Counter() {
  const [count, setCount] = useState(0);

  return (
    <div>
      <p>You clicked {count} times</p>
      <button onClick={() => setCount(count + 1)}>我是按钮</button>
    </div>
  );
}
```

<br/>
<br/>

<v-click>

1. 在点击 `按钮` 触发不停渲染的过程中，`count` 会“监听”状态的变化并自动更新吗?

</v-click>

---

```js
// 第一次渲染
function Counter() {
  const count = 0; // 从 useState() 返回
  // ...
  <p>You clicked {count} times</p>;
  // ...
}

// 点击按钮之后，Counter 被重新调用
function Counter() {
  const count = 1; // 从 useState() 返回
  // ...
  <p>You clicked {count} times</p>;
  // ...
}

// 点击按钮之后，Counter 被重新调用
function Counter() {
  const count = 2; // 从 useState() 返回
  // ...
  <p>You clicked {count} times</p>;
  // ...
}
```

---

## 事件处理函数呢？ [示例](https://codesandbox.io/s/compassionate-wood-qr3nj?file=/src/App.js)

<br />

```js
function Counter() {
  const [count, setCount] = useState(0);

  function handleAlertClick() {
    setTimeout(() => {
      alert("You clicked on: " + count);
    }, 3000);
  }

  return (
    <div>
      <p>You clicked {count} times</p>
      <button onClick={() => setCount(count + 1)}>Click me</button>
      <button onClick={handleAlertClick}>Show alert</button>
    </div>
  );
}
```

<v-click>

1. 输出的 count 是最新值 -- 事件处理函数在渲染过程中是不变的
2. 输出的 count 是调用 handleAlertClick 时的 count 值  -- 事件处理函数在每次渲染中会重新生成

</v-click>

---

```js {monaco}
// During first render
function Counter() {
  const count = 0; // Returned by useState()
  // ...
  function handleAlertClick() {
    setTimeout(() => {
      alert("You clicked on: " + count);
    }, 3000);
  }
  // ...
}

// After a click, our function is called again
function Counter() {
  const count = 1; // Returned by useState()
  // ...
  function handleAlertClick() {
    setTimeout(() => {
      alert("You clicked on: " + count);
    }, 3000);
  }
  // ...
}
```

---

# 每一次渲染都有它自己的…所有

在 Hooks 组件中，每次渲染的：

1. 状态 (state, props)
2. 事件处理函数
3. Effect
4. ...

都是独立的，它们仅仅属于定义它们的那次渲染

---

# useState

为函数组件添加状态

```js {all|1|3|4|all}
import { useState } from "react";
function Counter() {
  const [count, setCount] = React.useState(0);
  const increment = () => setCount(count + 1);
  return <button onClick={increment}>{count}</button>;
}
```

```js
const [state, setState] = useState(initialState);
```

参数：`initialState` 可以是任何值

返回值：`[属性, 修改属性的函数]` (数组 ❓)

```js {monaco}

```

---

# useState 使用注意点

## 1. 当属性为 引用数据类型 时，使用不可变数据结构

```js {monaco}
const Message = () => {
  const [messageObj, setMessage] = useState({ message: "" });
  return (
    <div>
      <input
        type="text"
        value={messageObj.message}
        onChange={(e) => {
          messageObj.message = e.target.value;
          setMessage(messageObj);
        }}
      />
    </div>
  );
};
```

<v-click>

React 使用 (`Object.is`)[https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/is] 来比较数据

</v-click>

---

## 2. `useState` 是"异步"更新的

```js {monaco}
function Counter() {
  const [count, setCount] = React.useState(0);
  const increment = () => {
    setCount(count + 1);
    setCount(count + 1);
    setCount(count + 1);
  };
  return <button onClick={increment}>{count}</button>;
}
```

<br />

<v-click>

为什么点击一次 count 加 1，而不是加 3 ？

</v-click>

<v-click>

1. 使用 函数式更新 `prevState => preSate + 1`  // 利用函数，接收旧值，进行更新

</v-click>

---

# 关于 batch update 批量更新

<br />
<br />
<br />


<v-click>

- 在 react 的 event handler (事件处理) 内部同步的多次 useState 会被 batch 为一次更新
- 在事件循环里的 useState 不会被批量更新
- 可以使用 unstable_batchedUpdates 来强制批量更新

</v-click>

---

## 3. Lazy initialization `useState`参数的惰性初始化

```js
// 执行一个开销很大的操作作为初始值  如 IO 操作
const [count, setCount] = React.useState(
  Number(window.localStorage.getItem("count"))
);

// 可以通过传入一个函数的方法来减少大开销执行的次数
const getInitialState = () => Number(window.localStorage.getItem("count"));
const [count, setCount] = React.useState(getInitialState);
```

性能优化的一种方式

[举个例子 🙋‍♂️🌰](https://codesandbox.io/s/eager-torvalds-z0egg?file=/src/App.js)

---

# useEffect & useLayoutEffect

处理副作用的函数，纯函数中和入参无关的处理值（数据获取，创建订阅，清理定时器等）

```js
useEffect(() => {
  effect;
  return () => {
    cleanup;
  };
}, [deps]);
```

<br />
<br />
<br />

<v-click>

1. useEffect 会在每次渲染后都执行吗？
2. React 何时清除 effect ? 在啥时执行 cleanup 函数

</v-click>

---

# useEffect 的执行时机

1. useEffect 会在每次渲染后都执行吗？

是的，默认情况下，它在第一次渲染之后和每次更新之后都会执行。React 保证了每次运行 effect 的同时，DOM 都已经更新完毕。

2. React 何时清除 effect？

React 会在组件卸载的时候执行清除操作。正如之前学到的，effect 在每次渲染的时候都会执行。这就是为什么 React 会在执行当前 effect 之前对上一个 effect 进行清除。

如果我们在 useEffect 的返回函数中使用 state, 我们要确保 deps 里包含它，否则只会取到 init 的值

---

# deps 依赖

useEffect 在没有设置第二个参数的时候，会在每次渲染的时候执行其回调

```js
const Example = () => {
  const [count, setCount] = useState(0);

  useEffect(() => {
    document.title = `You clicked ${count} times`;
  });

  return (
    <div>
      <p>You clicked {count} times</p>
      <button onClick={() => setCount(count + 1)}>Click me</button>
    </div>
  );
};
```

useEffect 有第二个参数，称为依赖数组，只有当依赖数组内的元素发生变化的时候，才会执行 useEffect 的回调。这么做就能够优化 effect 执行的次数。

---

# deps 依赖

<div class="flex mt-8">

<div v-click class="flex-grow w-1/2 pr-2">

需要依赖的 hooks 有：

- useEffect
- useCallback
- useMemo
- useImperativeHandle
- useLayoutEffect

都是参数的主体作为函数的方法

</div>

<div v-click class="flex-grow w-1/2 border-l border-vgreen pl-2">

deps 中的元素可以分为有基本类型、对象。细分为：

- 基本类型
- 函数
- 数据对象
- 混合对象 (由基本类型、函数、数据对象组成)

React 对依赖比较采用的是`Object.is`

</div>

</div>

---

## 函数作为依赖

<br/>

1. 使用 callback 包裹, 可能会导致依赖链
2. 纯化函数，让函数的依赖转换为函数参数
3. 进一步纯化，通过提取函数

<br/>
<br/>

## 数据对象最为依赖

<br/>

1. 分解为基本类型
2. 使用 `json.stringfy` 变为基本类型

---

# useLayoutEffect

99% 的情况下使用 useEffect, 两者的 api 相同，但是 useEffect 不适合执行修改 dom 的工作

差异
useEffect 是异步执行的，而 useLayoutEffect 是同步执行的。

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

---

# useRef & useImperativeHandle

Refs 提供了一种方式，允许我们访问 DOM 节点或在 render 方法中创建的 React 元素。

```js
const ref = useRef(initialValue);
// ref: { current: initialValue }
```

- 可以保存任何值

<v-click>

- 与直接在组件内部声明的 { current: ''} 对象的区别？

```js
const ref = useRef("");
const ref = { current: "" };
```

</v-click>

<v-click>

- 不会触发组件的重新渲染 (尽量不要在 UI 中使用，最好把改动动作放到 useState 前)

</v-click>

<v-click>

### 使用的地方

1. 在 Hooks 中作为一个全局变量使用。
2. 管理焦点（受控组件），文本选择或媒体播放
3. 触发强制动画
4. 控制 dom 节点 <!-- 小写的 jsx 标签 -->，控制子组件(配合 useImperativeHandle)

</v-click>

---

# useImperativeHandle

函数组件没有实例

```js
useImperativeHandle(ref，      // 父组件通过 ref 定义的引用变量
                    func,      // 子组件想要暴露给父组件的方法
                    [depts])   // deps
```

<v-click>

函数式组件是没有实例的，所以我们不能直接通过 ref 来调用子组件的方法。useImperativeHandle 可以让父组件获取并执行子组件内某些自定义函数(方法)。

本质上其实是子组件将自己内部的函数(方法)通过 useImperativeHandle 添加到父组件中 useRef 定义的对象中。

</v-click>

<v-click>

## 使用流程

1. useRef 创建引用变量
2. React.forwardRef 将引用变量传递给子组件
3. useImperativeHandle 将子组件内定义的函数作为属性，添加到父组件中的 ref 对象上。

[查看例子](https://codesandbox.io/s/condescending-turing-creon?file=/src/App.js)

</v-click>

---

# useMemo & useCallback

```js
export default function App() {
  const [count, setCount] = React.useState(0);
  const value = { name: 1 }; // 声明变量 value

  React.useEffect(() => {
    setCount(Math.random()); // 修改 count
    alert("render");
  }, [value]); // value 作为更新依赖
  return (
    <div className="App">
      <h1>Hello CodeSandbox</h1>
      <h2>Edit to see some magic happen!</h2>
    </div>
  );
}
```

App 渲染几次，value 被定义几次， alert 会被弹出几次?

<v-click>

无限循环，全都无限次。 组件渲染 → 创建一个新的 value -> useEffect 执行 → setCount 触发循环 → 组件渲染 → 创建一个新的 value -> useEffect 执行 → setCount 触发循环...

</v-click>

---

# useMemo & useCallback

解决方法:

```js {all|3,4,5,6|all}
export default function App() {
  const [count, setCount] = React.useState(0);
  const value = React.useMemo(() => {
    // 使用 useMemo 来缓存 value
    return { name: 1 };
  }, []);

  React.useEffect(() => {
    setCount(Math.random());
    alert("render");
  }, [value]);
  return (
    <div className="App">
      <h1>Hello CodeSandbox</h1>const [count, setCount] = useState(0);
      <h2>Edit to see some magic happen!</h2>
    </div>
  );
}
```

组件渲染 → 创建一个新的 value -> useEffect 执行 → setCount 触发循环 → 组件渲染 → 对比 Value 和前一次的引用地址一致 -> 结束

---

# useMemo & useCallback

useMemo 的意思就是：不要每次渲染都重新定义，而是我让你重新定义的时候再重新定义(第二个参数，依赖列表)。大家看到这里的依赖列表是空的，是因为 useMemo 里的回调函数确实没用到啥变量，如果有变量的话大家的 IDE 就会提醒加上依赖了。

这就是使用 useMemo 的原理，useMemo 适用于所有类型的值，加入这个值恰好是函数，那么用 useCallback 也可以。也就是说，useCallback 是一种特殊的 useMemo。

在这里再粗暴地给大家总结一下日常使用的场景：

如果你定义了一个变量，满足下面的条件就最好用 useMemo 或 useCallback 给包裹住：

1. 它不是状态，也就是说，不是用 useState 定义的(redux 中的状态实际上也是用 useState 定义的)
2. 它不是基本类型
3. 它会被放在 useEffect 的依赖列表里 || 自定义 hook 的返回值

---

# useCallback

1. 使用例子 1 useCallback 用来作为 Effect 依赖列表的缓存
<div class="flex">

<div v-click class="flex-grow w-1/2 pr-2">

```js
const fetchData = useCallback(() => {
  fetchAPI(a, b);
}, [a, b]);
```

</div>

<div v-click class="flex-grow w-1/2 border-l border-vgreen pl-2">

```js
useEffect(() => {
  fetchData();
}, [fetchData]);
```

</div>

</div>

2. 使用例子 2 useCallback + React.memo 当父组件传给子组件方法时
<div class="flex">

<div v-click class="flex-grow w-1/2 pr-2">

```js
const DemoUseCallback = ({ id }) => {
  const [number, setNumber] = useState(1);
  /* 此时usecallback的第一参数 (sonName)=>{ console.log(sonName) } 、*/
  const getInfo = useCallback(
    (sonName) => {
      console.log(sonName);
    },
    [id]
  );
  return (
    <div>
      {/* 点击按钮触发父组件更新 ，但是子组件没有更新 */}
      <button onClick={() => setNumber(number + 1)}>增加</button>
      <DemoChildren getInfo={getInfo} />
    </div>
  );
};
```

</div>

<div v-click class="flex-grow w-1/2 border-l border-vgreen pl-2">

```js
const DemoChildren = React.memo((props) => {
  /* 只有初始化的时候打印了 子组件更新 */
  console.log("子组件更新");
  useEffect(() => {
    props.getInfo("子组件");
  }, [props.getInfo]);
  return <div>子组件</div>;
});
```

</div>

</div>

<!-- ---

# useContext

解决组件之间的共享状态

```js
const MyContext = React.createContext("");
const value = useContext(MyContext);

<UserContext.Provider value={"chuanshi"}>
  <ComponentC />
</UserContext.Provider>;
```

找个例子 -->

<!-- ---

# useReducer

useReducer 是 React 提供的一个高级 Hook

https://juejin.cn/post/6844903869604986888 -->

---

# 自定义 Hooks

通过自定义 Hook，可以将组件逻辑提取到可重用的函数中。
注意点：状态都是在 hooks 中自己维护

例子

```js
const [form] = Form.useForm(); // dtd 2.0 中的表格交互
```

[userCounter Hooks](https://codesandbox.io/s/funny-snowflake-7u6rk?file=/src/App.js): `const [count, controlCount] = useCounter(10);`

[useModal Hooks(Dom Hooks)](https://codesandbox.io/s/practical-shadow-3tcvb?file=/src/App.js): `const [modal, toggleModal] = useModal()`

---

# 更多

## 优秀的第三方自定义 hooks 库

- [awsome-xxx 系列](https://github.com/rehooks/awesome-react-hooks)
- [ahooks 阿里沉淀的 Hooks 库](https://ahooks.gitee.io/zh-CN)

## 优秀的 Hooks 资料

- [React 官方文档](https://zh-hans.reactjs.org/docs/hooks-intro.html)
- [Dan Abramov 的博客](https://overreacted.io/zh-hans/a-complete-guide-to-useeffect/)

## 优秀的 Hooks 使用方法

- [Antd Table 的源码](https://github.com/ant-design/ant-design/tree/master/components/table/hooks)

---

# 作业

## 实现一个 useDebounce 自定义 State

<br>
<br>


```js
  const debouncedValue = useDebounce(value, time);
```

