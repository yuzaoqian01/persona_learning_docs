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

```toml
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

```json
{
  "name": "im-worker",
  "main": "src/index.ts",
  "compatibility_date": "2026-04-07",

  "durable_objects": { // DO配置
    "bindings": [
      {
        "name": "CONVERSATION",
        "class_name": "ConversationDO"
      }
    ]
  },

  "migrations": [ 
    {
      "tag": "v1",
      "new_sqlite_classes": ["ConversationDO"]
    }
  ],

  "kv_namespaces": [ // KV配置
    {
      "binding": "CACHE",
      "id": "你的 KV namespace id"
    }
  ],

  "r2_buckets": [ // R2 存储桶配置
    {
      "binding": "IM_BUCKET",
      "bucket_name": "你的 R2 bucket 名"
    }
  ],

  "d1_databases": [ // D1 配置
    {
      "binding": "DB",
      "database_name": "im-db",
      "database_id": "你的 D1 database id"
    }
  ]
}
```

代码里这样访问：

```ts
export interface Env {
  CONVERSATION: DurableObjectNamespace<ConversationDO>;
  CACHE: KVNamespace;
  IM_BUCKET: R2Bucket;
  DB: D1Database;
}
```

Worker 入口获取 DO：

```ts
const id = env.CONVERSATION.idFromName(conversationId);
const stub = env.CONVERSATION.get(id);
```

DO 内访问绑定：

```ts
export class ConversationDO extends DurableObject<Env> {
  constructor(ctx: DurableObjectState, env: Env) {
    super(ctx, env);
  }

  async saveLastMessage(conversationId: string, message: unknown) {
    await this.env.CACHE.put(
      `conversation:last:${conversationId}`,
      JSON.stringify(message),
      { expirationTtl: 3600 },
    );
  }

  async saveMessageToD1(message: {
    conversationId: string;
    seq: number;
    messageId: string;
    senderId: string;
    body: string;
    createdAt: number;
  }) {
    await this.env.DB.prepare(
      `
      INSERT INTO messages (
        conversation_id,
        seq,
        message_id,
        sender_id,
        body,
        created_at
      )
      VALUES (?, ?, ?, ?, ?, ?)
      `,
    )
      .bind(
        message.conversationId,
        message.seq,
        message.messageId,
        message.senderId,
        message.body,
        message.createdAt,
      )
      .run();
  }
}
```

创建资源命令大概是：

```cmd
npx wrangler kv namespace create CACHE
npx wrangler d1 create im-db
npx wrangler r2 bucket create im-bucket
```

然后把输出的 `id` / `database_id` 填回 `wrangler.jsonc`。

本地开发如果要预览：

```
npx wrangler dev
```

部署：

```
npx wrangler deploy
```
