上线前必做

1. 检测环境配置是否正确
2. 上线后验证一次所有功能
3. 如果是h5项目需要检查是否可滑动

```html
<title>FTG Quantify</title>
      <link rel="icon" href="./favicon.ico"></link>
      <meta charSet="utf-8" />
      <meta name="viewport" content="width=device-width, initial-scale=1.0, minimum-scale=1.0, maximum-scale=1.0, viewport-fit=cover, user-scalable=no" />
```

```css
// 安全区域
	constant(safe-area-inset-bottom)
	env(safe-area-inset-bottom)
```



4. 删除不需要的东西和文件
5. 项目配置做打包优化和兼容处理

```js
import { reactRouter } from "@react-router/dev/vite";
import { defineConfig } from "vite";
import tsconfigPaths from "vite-tsconfig-paths";
import legacy from '@vitejs/plugin-legacy'

export default defineConfig(({ mode }) => {
  const isProd = mode === "production";

  return {
    plugins: [reactRouter(), tsconfigPaths(),legacy({
      targets: ['defaults', 'not IE 11'],
    })],
    
    server: {
      host: true,
      port: 3000,
    },

    // 最关键：让 rollup 使用 es2015 
    build: {
      target: "es2015",          // ← 真正控制打包产物语法级别
      minify: "esbuild",
      sourcemap: false,
    },

    // esbuild 仅用于开发环境 & 少数处理
    esbuild: isProd
      ? {
          drop: ["console", "debugger"], // 只影响 esbuild 转换的部分，不影响最终语法
        }
      : {
          target: "es2015",
        }, 
  };
});

```

```json
{
  "name": "xxx",
  "private": true,
  "type": "module",
  "version": "1.0.0",
  "scripts": {
    "build": "react-router build",
    "build:dev": "react-router build --mode development",
    "dev": "react-router dev",
    "start": "react-router-serve ./build/server/index.js",
    "typecheck": "react-router typegen && tsc"
  },
  "dependencies": {
   
  },
  "devDependencies": {
    
  }
}
```

6. 项目read.me

````makefile
# 项目名称

xxx

## 项目简介

本项目为xxx

### 核心特性

- 首页xxx
- 国际化 语言 简体中文 英文
- 适配移动端 

### 技术栈

- 前端：react + vite + TypeScript
- tailwind3 shadcn/ui@2.3.0

## 环境要求

- Node.js >= 23.11.0
- npm >= 10.9.2
- pnpm >= 10.11.0
- 系统要求：Linux / macOS / Windows

## 安装

1. 克隆仓库：

```cmd
git clone "仓库地址"
```

2.安装依赖：

```cmd
npm install
```

3.配置环境变量：

env.development

4.启动项目：

```cmd
pnpm install
pnpm dev
```

## 使用

- 启动开发环境：`pnpm dev`
- 打开浏览器访问：`http://localhost:3000`

## 项目结构

```text
├─ app/          # 源码目录
│  ├─ components/  # react 组件
|  ├─assets/.  # 本地静态资源
│  ├─ pages/       # 页面
│  └─ store/       # 状态管理
├─ public/       # 静态资源
├─ tests/        # 测试用例
├─ .env.devlopment  # 环境变量示例
└─ README.md     # 本文件
```

## API 文档

- Swagger UI: 无
- API 说明：
- v1.0.0 需求文档  <https://www.kdocs.cn/l/ctNKCXSqP9dT>

## 版本历史

- v1.0.0 (2025-11-14)：初始版本

````

7. 打包依赖分析并优化