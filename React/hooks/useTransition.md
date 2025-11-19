# 1.`useTransition` 是什么

`useTransition` 用于 **标记“过渡更新”**，即告诉 React：

> 这部分更新不需要立刻渲染，可以先保持界面响应，然后再慢慢渲染。

特点：

- 不会阻塞用户交互（输入、点击）
- 常用于 **UI 更新大、慢的场景**
- 返回两个值：

```tsx
const [isPending, startTransition] = useTransition()
```

- `isPending` → 当前过渡是否还在进行
- `startTransition(callback)` → 标记 callback 内的更新为“过渡更新”

------

# 2. 基本用法

```tsx
import { useState, useTransition } from "react"

function Search() {
  const [query, setQuery] = useState("")
  const [results, setResults] = useState<string[]>([])
  const [isPending, startTransition] = useTransition()

  const handleChange = (e: React.ChangeEvent<HTMLInputElement>) => {
    const value = e.target.value
    setQuery(value) // 立即更新输入框值（用户体验）

    startTransition(() => {
      // 这个更新是“过渡更新”，可以延迟渲染
      const filtered = bigData.filter((item) =>
        item.toLowerCase().includes(value.toLowerCase())
      )
      setResults(filtered)
    })
  }

  return (
    <div>
      <input value={query} onChange={handleChange} />
      {isPending && <p>Loading...</p>}
      <ul>
        {results.map((item) => (
          <li key={item}>{item}</li>
        ))}
      </ul>
    </div>
  )
}
```

解释：

- 输入框立即响应
- 大数据过滤结果在后台渲染，不阻塞输入
- `isPending` 可以用来显示 Loading 提示

------

# 3. 使用场景

1. **搜索 / 过滤列表**
   - 用户输入 → 高性能渲染大列表
2. **分页 / 无限滚动**
   - 加载新数据时不阻塞 UI
3. **切换页面 / Tab / Drawer / Sheet**
   - 渲染复杂组件时保持界面响应
4. **跨链交易 / DApp UI**
   - 更新交易列表、状态卡片等大量数据时使用

------

# 4. 注意事项（大厂经验）

| 注意点                                  | 说明                                                |
| --------------------------------------- | --------------------------------------------------- |
| 过渡更新不能用于必须立即响应的状态      | 例如输入框值、点击状态等                            |
| 不要把 `useTransition` 用在全局状态更新 | 会影响整个应用性能                                  |
| 与 Suspense 搭配更强                    | `isPending` + `Suspense fallback` 可以显示 skeleton |
| 不要滥用                                | 小更新不需要过渡，过渡也会增加 React 调度开销       |
| startTransition 内可以触发多个 setState | React 会把它们合并为一个过渡更新                    |

------

# 5. React 18 的 Concurrent Mode 扩展

- Concurrent Mode 下，过渡更新可以被 **中断 / 延迟渲染**
- 保证 **用户输入不会卡顿**
- 适合 **大列表 / Table / 复杂动画 / 异步渲染**

------

# 6. 大厂常用模式

### 6.1 表单 + 列表过滤

```
const [query, setQuery] = useState("")
const [results, setResults] = useState([])
const [isPending, startTransition] = useTransition()

const handleSearch = (v: string) => {
  setQuery(v)
  startTransition(() => {
    setResults(heavyFilter(v))
  })
}
```

### 6.2 UI 更新延迟加载

```
<Button
  onClick={() => startTransition(() => setTab("bigComponent"))}
>
  切换大组件
</Button>
{isPending && <Skeleton />}
```

- 用户点击立即反馈（按钮按下）
- 大组件延迟渲染
- 使用 `isPending` 展示 loading skeleton

------

# 总结

1. `useTransition` 用于 **非紧急更新（过渡更新）**
2. 返回 `[isPending, startTransition]`
3. **紧急更新**：input value、按钮点击反馈
4. **过渡更新**：大列表渲染、复杂组件切换
5. 与 Suspense / Skeleton / Toast 搭配使用效果最佳
6. 注意不要滥用，性能消耗大