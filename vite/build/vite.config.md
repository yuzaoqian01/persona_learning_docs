```ts
// vite.config.ts

import { reactRouter } from "@react-router/dev/vite";
import { defineConfig } from "vite";
import tsconfigPaths from "vite-tsconfig-paths";

export default defineConfig(({ mode }) => {
  const isProd = mode === "production";
  return {
    plugins: [reactRouter(), tsconfigPaths()],
    server: {
      host: true,
      port: 3000,
    },
    esbuild: isProd
      ? {
          target: 'es2015',
          drop: ["console", "debugger"], // 在生产构建中去掉 console 和 debugger
        }
      : undefined,
  };
});


```

