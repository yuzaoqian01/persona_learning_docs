## 1. Fiber 是什么？

**Fiber 是 React 16 引入的核心架构重写**，它是 **React 内部用来描述组件及其状态、更新、子组件关系的数据结构**。

简单说：

> **Fiber 就是 React 的“轻量级任务单元”或“虚拟工作单元”，用于高效调度组件渲染。**

它类似于 **以前版本的 Stack Reconciler 栈**，但是 **可以中断和恢复渲染**，支持异步更新（Concurrent Mode）。

------



## 2. Fiber 的核心目的

React Fiber 重写的动机主要有三个：

1. **可中断渲染（Incremental Rendering）**
   - 旧版本同步渲染整个组件树，如果组件很多，渲染可能阻塞 UI，导致卡顿。
   - Fiber 允许 **拆分渲染任务**，把渲染分成小块，空闲时间执行，保证主线程不卡顿。
2. **优先级调度**
   - Fiber 可以给不同更新分配优先级（例如用户输入 > 网络请求 > 动画），保证高优先级更新先执行。
3. **支持异步渲染**
   - Fiber 为 React 的 Concurrent Mode 和 Suspense 打下基础，使 React 可以暂停、恢复或回退渲染。

------



## 3. Fiber 的数据结构

每个 Fiber 对象大致包含以下属性：

```
Fiber {
  type: Component,        // 组件类型
  key: string | null,     // 唯一标识
  stateNode: any,         // 真实 DOM 节点或 class 组件实例
  return: Fiber | null,   // 父 Fiber
  child: Fiber | null,    // 第一个子 Fiber
  sibling: Fiber | null,  // 下一个兄弟 Fiber
  memoizedState: any,     // 保存 hook 或 class state
  pendingProps: object,   // 新 props
  memoizedProps: object,  // 上一次渲染的 props
  effectTag: number,      // 标记需要执行的操作
  nextEffect: Fiber | null // effect 链表
}
```

**关系示意：**

```
Fiber(root)
 ├─ child -> Fiber(A)
 │    ├─ child -> Fiber(B)
 │    └─ sibling -> Fiber(C)
 └─ sibling -> Fiber(D)
```

- 每个 Fiber 代表组件的一个工作单元。
- Fiber 链表连接子组件和兄弟组件。
- `memoizedState` 用于保存组件状态和 hooks。

------



## 4. Fiber 与旧 Stack Reconciler 的区别

| 特性       | Stack Reconciler (旧) | Fiber (新)               |
| ---------- | --------------------- | ------------------------ |
| 渲染模式   | 同步                  | 异步 / 可中断            |
| 数据结构   | 栈                    | Fiber 链表               |
| 更新优先级 | 无                    | 支持优先级调度           |
| 渲染拆分   | 不支持                | 支持（增量渲染）         |
| 内存占用   | 少                    | 稍多，但可中断和恢复渲染 |

------



## 5. Fiber 工作流程概览

1. **调度阶段（Schedule Phase）**
   - React 接收到 setState 或 props 变化。
   - 创建新的 Fiber 树（工作链表）。
   - 标记需要更新的 Fiber，并分配优先级。
2. **渲染阶段（Render Phase / Reconciliation）**
   - React 遍历 Fiber 树，比较新旧 Fiber（diff）。
   - 生成 effect list（需要操作 DOM 的副作用）。
   - **可中断**，可以暂停渲染，把剩余任务放回队列。
3. **提交阶段（Commit Phase）**
   - 执行 effect list，把变更应用到真实 DOM。
   - 这是唯一 **同步且不可中断** 的阶段。

**流程图示意：**

```
更新触发
   │
调度阶段 ──> 创建 Fiber 树 ──> 遍历 Fiber ──> 构建 effect list
   │
提交阶段 ──> 执行 effect list ──> 更新真实 DOM
```

------



## 6. Fiber 与 Hooks 的关系

- 每个函数组件的 **useState、useReducer 等 hook 状态**都存储在对应 Fiber 的 `memoizedState` 上。
- Fiber 链表顺序对应 hook 调用顺序。
- 当 setState 调用时，Fiber 会标记需要更新，然后重新执行函数组件，更新 `memoizedState`。

------