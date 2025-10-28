## 1. 什么是 `memoizedState`？

在 **Fiber 对象**中，每个函数组件（或 class 组件）都会有一个 `memoizedState` 属性：

```
Fiber {
  ...
  memoizedState: Hook | null,
  ...
}
```

> `memoizedState` 就是 **Fiber 用来保存组件当前状态（state）和 hooks 信息的链表头**。

- 对于 **函数组件**，`memoizedState` 是 **hook 链表**的头结点。
- 对于 **class 组件**，`memoizedState` 保存的是 `this.state` 的缓存版本。

------



## 2. Hook 链表结构

函数组件中，`useState`、`useReducer`、`useEffect` 等 Hook 会生成链表节点：

```
Hook {
  memoizedState: any,   // 保存 hook 的状态值或 effect
  queue: UpdateQueue | null, // 更新队列，用于 setState
  next: Hook | null     // 下一个 hook
}
```

Fiber 的 `memoizedState` 就是这个链表的 **头结点**：

```
Fiber.memoizedState -> Hook(useState) -> Hook(useEffect) -> Hook(useState) -> null
```

> 每次渲染时，React 会根据 **hook 调用顺序**找到对应 hook 并更新 `memoizedState`。

------



## 3. `memoizedState` 与 useState 的关系

### 3.1 初次渲染

```
function Counter() {
  const [count, setCount] = useState(0);
}
```

- React 创建一个 Fiber。

- Fiber.memoizedState 指向 Hook 节点：

  ```
  {
    memoizedState: 0,  // 当前状态
    queue: [],         // 更新队列
    next: null
  }
  ```

------

### 3.2 更新状态

```
setCount(c => c + 1)
```

- React 将更新放入 **queue**：

  ```
  hook.queue.push(action)
  ```

- 在下一次渲染阶段：

  - React 遍历 hook 链表。

  - 对每个 hook 执行队列中的更新：

    ```
    hook.memoizedState = typeof action === 'function' ? action(hook.memoizedState) : action
    ```

- 更新后的状态写回 `memoizedState`，并清空 queue。

------



## 4. 为什么叫 “memoizedState”？

“memoized” 的意思是 **记忆化缓存**。

- 它保存了 **上一次渲染的状态**，用于下一次渲染的比较和更新。

- 可以理解为：

  > Fiber memoizes the state of hooks to ensure consistent updates across renders.

------



## 5. 总结

1. **`memoizedState` 是 Fiber 上存储 hook 状态的头指针**。
2. **它是一个链表结构**，每个 hook 都是一个节点，包含：
   - 当前状态值 `memoizedState`
   - 更新队列 `queue`
   - 下一个 hook `next`
3. **`setState` 的更新流程**：
   - 将更新放入队列
   - Fiber 在下次渲染遍历 hook 链表
   - 使用队列计算新的 `memoizedState`
4. **保证状态稳定**：
   - Hook 调用顺序固定
   - 每次渲染拿到的状态都是上一轮的 `memoizedState`

------