## 一、Tailwind v4 默认用到的“危险特性”

Tailwind v4 会大量依赖：

| 特性               | 老环境支持 |
| ------------------ | ---------- |
| CSS Variables      | ❌ 老 iOS   |
| `:where()`         | ❌          |
| CSS Nesting        | ❌          |
| `@layer`           | ❌          |
| `color-mix()`      | ❌          |
| logical properties | ❌          |

👉 **不处理 = 一定翻车**

------

## 二、大厂通用方案（强烈推荐）

### ✅ 核心思路

> **Tailwind v4 + PostCSS preset-env + Autoprefixer**

让 **现代 CSS → 老 CSS**

------

## 三、必须的 PostCSS 配置（重点）

### `postcss.config.js`

```
module.exports = {
  plugins: {
    'postcss-preset-env': {
      stage: 2,
      features: {
        'nesting-rules': true
      }
    },
    autoprefixer: {}
  }
}
```

👉 这一步是 **兼容的核心**

------

## 四、Tailwind 配置（必须限制）

### `tailwind.config.js`

```
export default {
  content: ['./index.html', './src/**/*.{js,ts,jsx,tsx}'],
  theme: {
    extend: {}
  },
  corePlugins: {
    preflight: true
  },
  future: {
    hoverOnlyWhenSupported: true
  }
}
```

### ⚠️ 注意点

- **不要**用实验性插件
- **不要**自己写 CSS nesting
- **少用 arbitrary value**：`w-[calc(...)]`

------

## 五、浏览器目标（非常关键）

### `.browserslistrc`

```
> 0.5%
last 2 versions
iOS >= 12
Android >= 7
not dead
```

👉 Autoprefixer / preset-env 都靠这个工作

------

## 六、Vite 项目（老环境必须加）

### 1️⃣ 加 legacy 插件

```
npm i @vitejs/plugin-legacy
import legacy from '@vitejs/plugin-legacy'

export default {
  plugins: [
    legacy({
      targets: ['iOS >= 12', 'Android >= 7'],
      additionalLegacyPolyfills: ['regenerator-runtime/runtime']
    })
  ]
}
```

------

### 2️⃣ 明确引入 polyfill（钱包 WebView 必须）

```
import 'core-js/stable'
import 'regenerator-runtime/runtime'
```

------

## 七、Tailwind v4 使用建议（老环境安全写法）

### ✅ 推荐

```
<div class="flex items-center justify-between p-4">
<div class="bg-[#fff] text-[14px]">
```

------

### ❌ 慎用 / 禁用

```
<!-- nesting -->
<div class="[&_.child]:text-red-500"></div>

<!-- where -->
<div class="group-hover:opacity-50"></div>

<!-- css var 强依赖 -->
<div class="text-[var(--color)]"></div>
```

------

## 八、钱包 / WebView 实战踩坑（你一定会遇到）

### iOS 钱包内 WebView

| 问题                  | 解决             |
| --------------------- | ---------------- |
| flex gap 不生效       | 用 margin        |
| position: sticky 失效 | JS 兜底          |
| vh 不准               | 用 JS 计算       |
| safe-area 问题        | env() + fallback |