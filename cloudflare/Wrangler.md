#  Wrangler 是什么？

Wrangler = Cloudflare Workers 的开发与部署工具。

你可以用它来：

- 本地开发 Workers（自动热加载）
- 部署到 Cloudflare
- 管理 KV / D1 / R2 等 Cloudflare 资源
- 预览 Pages Functions
- 配置环境变量、Secrets
- 管理多个环境（staging / production）

------

# 安装 Wrangler

```
npm install -g wrangler
```

或使用 pnpm：

```cmd
pnpm add -g wrangler
```

------

# 初始化一个 Workers 项目

```
wrangler init my-worker
cd my-worker
```

使用模板：

```
wrangler init --template cloudflare/workers-ai
```

------

# 本地开发

```
wrangler dev
```

会开启本地模拟环境（同 Cloudflare 上运行环境一致）。

------

# 部署到 Cloudflare

```
wrangler deploy
```

部署到特定环境：

```
wrangler deploy --env production
```

------

# 管理 Secrets（环境变量）

添加 secret：

```
wrangler secret put API_KEY
```

查看已绑定：

```
wrangler secret list
```

------

# KV / D1 / R2 相关命令

### 创建 KV 命名空间

```
wrangler kv:namespace create "MY_NAMESPACE"
```

### 创建 D1 数据库

```
wrangler d1 create my-database
```

本地执行 SQL：

```
wrangler d1 execute my-database --local --file=./schema.sql
```

### R2 存储桶

```
wrangler r2 bucket create my-bucket
```

------

# wrangler.toml 样例

```
name = "my-worker"
main = "src/index.ts"
compatibility_date = "2024-01-01"

[vars]
ENV = "dev"

[kv_namespaces]
  binding = "MY_KV"
  id = "xxxxxxxxxxxxx"
```

------

