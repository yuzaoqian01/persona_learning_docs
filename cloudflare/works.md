# 安装

```cmd
pnpm create cloudflare@latest my-first-worker
```

```css

wrangler.jsonc // wrangler 配置文件
index.js //最小的worker
package.json // node 依赖文件

```

# 本地预览worker

```cmd
npx wrangler dev
```

# 部署项目

```cmd
npx wrangler deploy
```

# 创建数据库

```cmd
npx wrangler d1 create members-db
```

# 验证当前生效配置

```cmd
npx wrangler whoami --verbose
```



```json
compatibility_flags = ["nodejs_compat"]

[[d1_databases]]
binding = "DB" //数据库
database_name = "members-db" //数据库名称
database_id = "6471ec9f-777f-4e4b-b50e-d6d9f3344577" // 数据库绑定id
```

```json
/**
 * For more details on how to configure Wrangler, refer to:
 * https://developers.cloudflare.com/workers/wrangler/configuration/
 */
{
	"$schema": "node_modules/wrangler/config-schema.json",
	"name": "worker-01",
	"main": "src/index.ts",
	"compatibility_date": "2025-12-05",
	"observability": {
		"enabled": true
	},
	"compatibility_flags": [
		"nodejs_compat"
	],
	"d1_databases": [
		{
			"binding": "members_db",
			"database_name": "members-db",
			"database_id": "6471ec9f-777f-4e4b-b50e-d6d9f3344577"
		}
	],
	/**
	 * Smart Placement
	 * Docs: https://developers.cloudflare.com/workers/configuration/smart-placement/#smart-placement
	 */
	// "placement": { "mode": "smart" }
	/**
	 * Bindings
	 * Bindings allow your Worker to interact with resources on the Cloudflare Developer Platform, including
	 * databases, object storage, AI inference, real-time communication and more.
	 * https://developers.cloudflare.com/workers/runtime-apis/bindings/
	 */
	/**
	 * Environment Variables
	 * https://developers.cloudflare.com/workers/wrangler/configuration/#environment-variables
	 */
	// "vars": { "MY_VARIABLE": "production_value" }
	/**
	 * Note: Use secrets to store sensitive data.
	 * https://developers.cloudflare.com/workers/configuration/secrets/
	 */
	/**
	 * Static Assets
	 * https://developers.cloudflare.com/workers/static-assets/binding/
	 */
	// "assets": { "directory": "./public/", "binding": "ASSETS" }
	/**
	 * Service Bindings (communicate between multiple Workers)
	 * https://developers.cloudflare.com/workers/wrangler/configuration/#service-bindings
	 */
	// "services": [{ "binding": "MY_SERVICE", "service": "my-service" }]
}
```



| 字段              | 说明                                         |
| ----------------- | -------------------------------------------- |
| **binding**       | 你在 Worker 代码中使用的变量名（环境变量名） |
| **database_name** | D1 数据库在 Cloudflare Dashboard 的名称      |
| **database_id**   | 数据库的唯一 ID                              |



```json
{
  // ✨ Wrangler 配置文件的 Schema，用于 VS Code 自动补全和报错提示
  "$schema": "node_modules/wrangler/config-schema.json",

  // 🏷️ Worker 的名称，用于 Cloudflare workers.dev 子域名
  "name": "my-worker-project",

  // 📌 主入口文件（TypeScript 或 JavaScript）
  // 支持 ESM（export default），并且不支持 Node.js require()
  "main": "src/index.ts",

  // 📅 锁定当前兼容性日期，确保未来 Cloudflare 升级不会破坏你的代码
  "compatibility_date": "2025-12-05",

  // ⚙️ 启用 Cloudflare Observability（日志 / 性能分析）
  "observability": {
    "enabled": true
  },

  // 🚀 开启 Node.js 兼容层（重要！用于 web3、加密库、Buffer 等）
  "compatibility_flags": [
    "nodejs_compat"
  ],

  // 📦 构建设置，用于 Typescript → JS 以及 Bun/Vite 项目
  "build": {
    // Wrangler 内置 esbuild，不需要额外工具
    "command": "npm run build",
    // 输入与输出
    "cwd": "."
  },

  // 🔑 公开环境变量（在客户端可见）
  "vars": {
    "PUBLIC_APP_VERSION": "1.0.0"
  },

  // 🔒私有环境变量（仅 Worker 内部可见）
  // 适用于 API Keys、密钥等
  "env": {
    "production": {
      "vars": {
        "API_KEY": "xxxx-prod"
      }
    },
    "dev": {
      "vars": {
        "API_KEY": "xxxx-dev"
      }
    }
  },

  // 🗄️ 绑定 Cloudflare D1 数据库
  "d1_databases": [
    {
      // Worker 里访问的变量名：env.DB
      "binding": "DB",
      "database_name": "my-d1-db",
      "database_id": "11112222-3333-4444-5555-666677778888"
    }
  ],

  // 🔐 绑定环境密钥（如 JWT 秘钥, encryption key）
  "encrypted_secrets": [
    "JWT_SECRET"
  ],

  // 🗃️ KV Namespace 绑定（用于键值存储）
  "kv_namespaces": [
    {
      "binding": "MY_KV",
      "id": "abcd1234abcd1234abcd1234abcd1234"
    }
  ],

  // 🪣 R2 云储存
  "r2_buckets": [
    {
      "binding": "MY_BUCKET",
      "bucket_name": "my-r2-bucket"
    }
  ],

  // 📬 Queue（消息队列）
  "queues": {
    "consumers": [
      {
        "queue": "my-queue",
        "max_batch_size": 10
      }
    ],
    "producers": [
      {
        "binding": "MY_QUEUE"
      }
    ]
  },

  // 🧪 本地开发设置（端口、Persist 等）
  "dev": {
    // 本地 Worker Dev Server 端口
    "port": 8787,
    // 是否持久化 D1 / KV 数据
    "persist": true,
    // 监视文件变更并自动 reload
    "watch": true
  },

  // ⚡ 发布到 Cloudflare 时的设置
  "routes": [
    {
      // 自定义域名
      "pattern": "api.example.com/*",
      "zone_name": "example.com"
    }
  ]
}

```



```toml
# Worker 名称
name = "my-worker"

# 入口文件
main = "src/index.ts"

# 锁定兼容性日期
compatibility_date = "2025-12-05"

# Node.js 兼容模式
compatibility_flags = ["nodejs_compat"]

# 可观测性
[observability]
enabled = true

# ----------- 数据库、KV、R2 -----------

# D1 数据库绑定
[[d1_databases]]
binding = "DB"
database_name = "my-db"
database_id = "1234abcd-5678-ef00-9999-0000abcdefff"

# KV Namespace
[[kv_namespaces]]
binding = "MY_KV"
id = "abcdabcdabcdabcdabcdabcdabcdabcd"

# R2 bucket
[[r2_buckets]]
binding = "MY_BUCKET"
bucket_name = "my-bucket"

# ----------- 队列 Queue -----------

[queues]
  [[queues.producers]]
  binding = "MY_QUEUE"

  [[queues.consumers]]
  queue = "my-queue"
  max_batch_size = 10

# ----------- 环境变量 -----------

[vars]
PUBLIC_VERSION = "1.0.0"

# ----------- 路由 -----------

[[routes]]
pattern = "api.example.com/*"
zone_name = "example.com"

# ----------- 开发环境 -----------

[dev]
port = 8787
persist = true
watch = true

```



```sql
//schemas/schema.sql
DROP TABLE IF EXISTS members;
CREATE TABLE IF NOT EXISTS members (
  id INTEGER PRIMARY KEY AUTOINCREMENT,
  name TEXT NOT NULL,
  email TEXT NOT NULL UNIQUE,
  joined_date TEXT NOT NULL
);

-- Insert sample data
INSERT INTO members (name, email, joined_date) VALUES
  ('Alice Johnson', 'alice@example.com', '2024-01-15'),
  ('Bob Smith', 'bob@example.com', '2024-02-20'),
  ('Carol Williams', 'carol@example.com', '2024-03-10');
```

```ts
import { env } from "cloudflare:workers";
import { httpServerHandler } from "cloudflare:node";
import express from "express";

const app = express();

// Middleware to parse JSON bodies
app.use(express.json());

// Health check endpoint
app.get("/", (req, res) => {
  res.json({ message: "Express.js running on Cloudflare Workers!" });
});

// GET all members
app.get('/api/members', async (req, res) => {
  try {
    const { results } = await env.members_db.prepare('SELECT * FROM members ORDER BY joined_date DESC').all();

    res.json({ success: true, members: results });
  } catch (error) {
    res.status(500).json({ success: false, error: 'Failed to fetch members' });
  }
});

// GET a single member by ID
app.get('/api/members/:id', async (req, res) => {
  try {
    const { id } = req.params;

    const { results } = await env.members_db.prepare('SELECT * FROM members WHERE id = ?').bind(id).all();

    if (results.length === 0) {
      return res.status(404).json({ success: false, error: 'Member not found' });
    }

    res.json({ success: true, member: results[0] });
  } catch (error) {
    res.status(500).json({ success: false, error: 'Failed to fetch member' });
  }
});


// POST - Create a new member
app.post("/api/members", async (req, res) => {
  try {
    const { name, email } = req.body;

    // Validate input
    if (!name || !email) {
      return res.status(400).json({
        success: false,
        error: "Name and email are required",
      });
    }

    // Basic email validation (simplified for tutorial purposes)
    // For production, consider using a validation library or more comprehensive checks
    if (!email.includes("@") || !email.includes(".")) {
      return res.status(400).json({
        success: false,
        error: "Invalid email format",
      });
    }

    const joined_date = new Date().toISOString().split("T")[0];

    const result = await env.members_db.prepare(
      "INSERT INTO members (name, email, joined_date) VALUES (?, ?, ?)"
    )
      .bind(name, email, joined_date)
      .run();

    if (result.success) {
      res.status(201).json({
        success: true,
        message: "Member created successfully",
        id: result.meta.last_row_id,
      });
    } else {
      res
        .status(500)
        .json({ success: false, error: "Failed to create member" });
    }
  } catch (error: any) {
    // Handle unique constraint violation
    if (error.message?.includes("UNIQUE constraint failed")) {
      return res.status(409).json({
        success: false,
        error: "Email already exists",
      });
    }
    res.status(500).json({ success: false, error: "Failed to create member" });
  }
});

app.put("/api/members/:id", async (req, res) => {
  try {
    const { id } = req.params;
    const { name, email } = req.body;

    // Validate input
    if (!name && !email) {
      return res.status(400).json({
        success: false,
        error: "At least one field (name or email) is required",
      });
    }

    // Basic email validation if provided (simplified for tutorial purposes)
    // For production, consider using a validation library or more comprehensive checks
    if (email && (!email.includes("@") || !email.includes("."))) {
      return res.status(400).json({
        success: false,
        error: "Invalid email format",
      });
    }

    // Build dynamic update query
    const updates: string[] = [];
    const values: any[] = [];

    if (name) {
      updates.push("name = ?");
      values.push(name);
    }
    if (email) {
      updates.push("email = ?");
      values.push(email);
    }

    values.push(id);

    const result = await env.members_db.prepare(
      `UPDATE members SET ${updates.join(", ")} WHERE id = ?`
    )
      .bind(...values)
      .run();

    if (result.meta.changes === 0) {
      return res
        .status(404)
        .json({ success: false, error: "Member not found" });
    }

    res.json({ success: true, message: "Member updated successfully" });
  } catch (error: any) {
    if (error.message?.includes("UNIQUE constraint failed")) {
      return res.status(409).json({
        success: false,
        error: "Email already exists",
      });
    }
    res.status(500).json({ success: false, error: "Failed to update member" });
  }
});

// DELETE - Delete a member
app.delete("/api/members/:id", async (req, res) => {
  try {
    const { id } = req.params;

    const result = await env.members_db.prepare("DELETE FROM members WHERE id = ?")
      .bind(id)
      .run();

    if (result.meta.changes === 0) {
      return res
        .status(404)
        .json({ success: false, error: "Member not found" });
    }

    res.json({ success: true, message: "Member deleted successfully" });
  } catch (error) {
    res.status(500).json({ success: false, error: "Failed to delete member" });
  }
}); 

app.listen(3000);
export default httpServerHandler({ port: 3000 });
```



# 对d1数据执行

```cmd 
npx wrangler d1 execute members-db --file=./schemas/schema.sql
```



```cmd
 /* 执行 typegen 命令为您的 Worker 环境生成类型定义 */
npm run cf-typegen
```

# 对远程数据做操作

```cmd
npx wrangler d1 execute members-db --remote --file=./schemas/schema.sql
```

# 部署

```cmd
npm run deploy
```

