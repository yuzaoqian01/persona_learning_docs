## 安装

是一个打包依赖分析库

```shell
npm install --save-dev rollup-plugin-visualizer
```

## 使用

```shell
// es
import { visualizer } from "rollup-plugin-visualizer";
// or
// cjs
const { visualizer } = require("rollup-plugin-visualizer");
```

```ts
//vite.config.ts
import { defineConfig, type PluginOption } from "vite";

export default defineConfig({
  plugins: [visualizer() as PluginOption],
});
```

