#  React 编码规范文档

------

## 1. 基础编码规范

- 组件命名：PascalCase，例如 `StakingButton`
- 变量和函数命名：camelCase，例如 `handleSubmit`
- Hooks 命名必须以 `use` 开头，例如 `useFetchData`
- 单文件长度不超过 500 行，复杂组件应拆分
- 避免匿名内联函数导致无限重渲染
- 循环中创建组件禁止
- 条件语句中禁止调用 Hooks
- map 必须指定 key，避免使用 index（特殊情况除外）

------

## 2. 状态管理

- UI 状态使用 `useState`（例如 loading 状态）
- 逻辑锁使用 `useRef`（防重复点击、防重入）
- 复杂状态使用 `useReducer`
- 昂贵计算使用 `useMemo`
- 回调传子组件使用 `useCallback`（仅在必要时）
- 跨页面共享状态使用 Zustand 或 Redux Toolkit

------

## 3. 按钮事件与防重入

```ts
const isRunningRef = useRef(false);
const [loading, setLoading] = useState(false);

const handleClick = async () => {
  if (isRunningRef.current) return;
  isRunningRef.current = true;
  setLoading(true);

  try {
    await apiCall();
  } catch (err) {
    toast.error(err.message);
  } finally {
    isRunningRef.current = false;
    setLoading(false);
  }
};
```

- UI loading 与逻辑锁分离
- useRef 防止重复触发
- useState 控制按钮 UI

------

## 4. Hooks 使用规范

- useEffect 只做副作用，避免业务逻辑或异步计算
- 异步逻辑在 useEffect 内用内部 async 函数封装
- 自定义 Hooks 保持纯逻辑，无副作用

------

## 5. 网络请求规范

- 推荐使用 React Query 或 SWR
- API 必须封装到 `/api` 或 `/services`
- 所有请求必须捕获异常
- 返回结果必须类型明确

------

## 6. 组件设计规范

- 单一职责，每个组件只做一件事
- 可复用组件统一放 `/components`
- props 分类：必填前置，可选后置，样式和事件使用 restProps
- 避免组件内部直接请求网络
- 避免内部维护大量逻辑状态

------

## 7. 性能优化

- useMemo 优化昂贵计算
- useCallback 优化回调传递
- React.memo 优化列表或大型子组件
- 大于 200 条列表必须使用 react-window 或 react-virtualized

------

## 8. 异常处理

- 网络异常必须捕获并显示 toast
- 内部错误必须 console.error
- 关键 UI 使用 ErrorBoundary 捕获

------

## 9. 表单规范

- 必须使用受控组件
- 数字输入必须使用 inputMode="decimal"
- 提交必须节流 + 防重入锁
- 输入校验必须提供错误提示

------

## 10. 可维护性规范

- 通用逻辑抽成 hooks，例如 useStake、useLock、useFetch
- 不可重复逻辑放 service 层
- API 类型统一放 `/types/api.ts`
- 所有 hook / service 必须有完整注释

------

## 11. 文件结构

```
src/
  components/
  hooks/
  pages/
  services/
  api/
  store/
  styles/
  utils/
  types/
```

------

## 12. UI 交互规范

- 按钮必须有 loading 状态
- Toast 统一样式和位置
- 表单必须有输入校验
- 列表渲染必须有稳定 key

------

## 13. 常见错误避免

- 按钮 async 直接调用导致重复触发
- useEffect 中依赖写不完整导致死循环
- 使用 index 作为 key 导致列表更新异常
- 组件内执行复杂逻辑导致卡顿
- ref 被滥用为状态（应使用 state）

------

## 14. 总结

- **行为锁 → useRef**
- **UI 状态 → useState**
- **网络请求 → React Query / API 封装**
- **逻辑分离 → hooks + services**
- **性能优化 → memo / callback / useMemo**
- **文件结构 → 统一规范**
- **异常处理 → toast + console**
- **表单 → controlled + 校验**
- **列表 → key 稳定**
- **条件渲染 → 简单清晰**

------

此文档可作为团队 React 项目的统一编码规范，并参考阿里、美团、字节、腾讯等大厂实践。