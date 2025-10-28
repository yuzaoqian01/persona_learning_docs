## 1、基础用法

```tsx
import React, { useState } from 'react';

function Counter() {
  const [count, setCount] = useState(0); // 初始化状态为 0
  
  // 自己理解的执行方法
  funtion Increment(){
    setCount(count=>count + 1);
  }

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setCount(count + 1)}>Increment</button>//注意后期如果写了方法不能放方法名
      <button onClick={()=> Increment()}>Increment</button>
    </div>
  );
}

export default Counter;

```



## 2、支持任意类型

```tsx
const [name, setName] = useState("Alice"); // 字符串
const [isActive, setIsActive] = useState(false); // 布尔值
const [items, setItems] = useState([]); // 数组
const [user, setUser] = useState({ id: 1, name: "Bob" }); // 对象

```

**注意对象或数组更新：**
 直接修改对象/数组会出问题，需要创建新副本：

```tsx
setUser({ ...user, name: "Charlie" }); // 不直接修改 user，而是创建新对象
setItems([...items, "new item"]); // 不直接 push，而是创建新数组
```

------



## 3、惰性初始化（Lazy Initialization）

如果状态初始值计算复杂，可以传入函数，**只在初始渲染时计算一次**：

```
const [value, setValue] = useState(() => {
  console.log("计算初始值");
  return expensiveComputation();
});
```

> 每次重新渲染时不会再执行 `expensiveComputation`，只在第一次渲染时执行。



## 4. 状态更新的特殊情况

### 4.1 基于旧状态更新

如果新状态依赖于旧状态，最好使用 **函数形式**：

```
setCount(prevCount => prevCount + 1);
```

这样可以避免异步更新导致的问题，例如：

```
<button onClick={() => setCount(count + 1)}>Increment</button>
<button onClick={() => setCount(count + 1)}>Increment</button>
```

如果连续两次使用 `count + 1`，可能只加一次；用函数形式就没问题。

------

### 4.2 批量更新

React 18 及以上版本中，**在事件处理器、生命周期中多次调用 `setState` 会批量合并**：

```
setCount(c => c + 1);
setCount(c => c + 1); // 最终增加 2
```

------



## 5. 使用数组或对象状态的技巧

### 5.1 对象状态

```
const [user, setUser] = useState({ name: "", age: 0 });

setUser(prev => ({ ...prev, name: "Alice" })); // 保留 age
```

### 5.2 数组状态

```
const [items, setItems] = useState([1, 2, 3]);

// 添加元素
setItems(prev => [...prev, 4]);

// 删除元素
setItems(prev => prev.filter(item => item !== 2));
```

------



## 6. `useState` 与闭包陷阱

在事件或异步回调中，状态可能会被“捕获”，导致拿到的是旧值：

```
const [count, setCount] = useState(0);

function handleClick() {
  setTimeout(() => {
    console.log(count); // 这里拿到的可能是旧 count
    setCount(count + 1);
  }, 1000);
}
```

**解决方法：使用函数形式更新**

```
setCount(prevCount => prevCount + 1);
```

------

## 7. 总结进阶要点

1. **函数更新**避免闭包陷阱：`setState(prev => newValue)`
2. **惰性初始化**避免不必要的计算：`useState(() => expensiveComputation())`
3. **对象/数组状态**必须创建新副本，不要直接修改原状态
4. **批量更新**React 会自动合并同一事件内的状态更新
5. **局部状态与组件重渲染**：`useState` 的状态是组件局部的，改变状态会触发组件重新渲染。

------



# 1. `useState` 的核心原理

React 的 `useState` 并不是普通的变量，它背后依赖 **React Fiber 架构**和 **hooks 的链表存储机制**来管理状态。

## 核心点：

1. **状态存储在 Fiber 上**
    每个函数组件在渲染时会对应一个` Fiber `对象，Fiber 上有一个 `memoizedState` 链表，用来保存组件的所有 hooks 状态。个人理解所以这里我理解他是非全局的状态 只是在这个函数组件内有状态 无法应用到全局 除非使用useContext?或者状态库啥的

1. **Hooks 调用顺序固定**

   - React 通过调用顺序来识别每个 hook 对应的状态位置。
   - 这就是为什么 hooks 必须 **在组件最顶层调用**，不能在条件语句或循环里调用。

2. **setState 会触发调度**

   - `setState` 不直接修改状态变量，而是生成一个 **更新队列**。
   - React 调度器会根据队列重新渲染组件，并更新 Fiber 上的状态。

   

# 2. `useState` 内部数据结构

可以抽象成：

```js
// Hook 链表节点
function Hook(initialState) {
  this.memoizedState = initialState; // 当前状态
  this.queue = null;                  // 更新队列
  this.next = null;                   // 下一个 hook
}
```

组件的 Fiber 结构大致是：

```
Fiber {
  memoizedState -> Hook1 -> Hook2 -> Hook3 -> null
}
```

每个 Hook 保存它自己的状态和更新队列。

------



# 3. `setState` 的实现原理

`setState` 实际上做三件事：

1. **将更新加入 hook 的更新队列**

   ```js
   hook.queue.push(action)
   ```

   这里的 `action` 可以是新值或者函数 `(prevState) => newState`。

2. **触发重新渲染调度**
    React 会将当前组件标记为“需要更新”，并在下一个渲染周期重新执行函数组件。

3. **在渲染阶段处理队列**
    React 遍历 hook 的更新队列，计算最终状态：

   ```js
   let newState = hook.memoizedState;
   hook.queue.forEach(action => {
     newState = typeof action === 'function' ? action(newState) : action;
   });
   hook.memoizedState = newState;
   hook.queue = [];
   ```

------



# 4. 简化版 `useState` 自实现示例

下面是一个非常基础的 `useState` 模拟实现：

```js
let hooks = []; // 全局存储 hooks
let currentHook = 0;

function useState(initialValue) {
  const hookIndex = currentHook;

  // 第一次渲染时初始化
  if (!hooks[hookIndex]) {
    hooks[hookIndex] = {
      state: initialValue,
      queue: []
    };
  }

  const setState = (action) => {
    const hook = hooks[hookIndex];
    // 支持函数更新
    const newState = typeof action === "function" ? action(hook.state) : action;
    hook.state = newState;
    render(); // 重新渲染组件
  };

  const state = hooks[hookIndex].state;

  currentHook++;
  return [state, setState];
}

// 模拟组件渲染
function render() {
  currentHook = 0; // 重置索引
  App();           // 调用函数组件
}

// 示例组件
function App() {
  const [count, setCount] = useState(0);
  console.log("render", count);

  // 模拟点击事件
  if (count < 3) setTimeout(() => setCount(c => c + 1), 1000);
}

render();
```

**解释：**

1. `hooks` 数组模拟 Fiber 上的 hook 链表。
2. `currentHook` 模拟 React 内部的 hook 调用顺序。
3. `setState` 会处理函数式更新并触发重新渲染。
4. 每次渲染前，`currentHook` 都会重置。

运行这个代码，你会看到 `render 0`, `render 1`, `render 2`, … 逐渐更新。

------



# 5. 关键原理总结

1. **状态存储**
    React 把每个 hook 的状态保存在 Fiber 上，通过链表或者数组管理。
2. **调用顺序决定状态位置**
    hook 的“槽位”由调用顺序确定，保证每次渲染对应正确的状态。
3. **更新队列**
    setState 并不立即改变状态，而是加入更新队列，等待下次渲染处理。
4. **函数式更新防闭包问题**
    setState 可以传函数 `(prev) => next`，保证异步更新不会丢失状态。

------

