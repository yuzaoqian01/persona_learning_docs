### 一、Durable Object = Cloudflare 提供的“全球唯一单线程状态实例”

```ts
import { DurableObject } from "cloudflare:workers";

/**
 * Durable Object（持久化对象）
 *
 * Cloudflare Durable Objects 是一种“有状态”的 Worker。
 * 普通 Worker 默认是无状态的，而 Durable Object 可以：
 *
 * 1. 保存内存状态
 * 2. 保证同一个 ID 的请求始终进入同一个实例
 * 3. 适合做：
 *    - 聊天室
 *    - WebSocket 长连接
 *    - 游戏房间
 *    - 分布式锁
 *    - 实时协同
 *    - 计数器
 *    - Session
 *
 * 官方文档：
 * [Cloudflare Durable Objects](https://developers.cloudflare.com/durable-objects/?utm_source=chatgpt.com)
 */

/**
 * Durable Object 类
 *
 * 每一个 Durable Object 实例，都类似一个“单线程小服务器”
 */
export class MyDurableObject extends DurableObject<Env> {
	/**
	 * 构造函数
	 *
	 * 什么时候执行：
	 * - 第一次访问这个 DO 实例时执行
	 * - 或实例被休眠后再次唤醒时执行
	 *
	 * @param ctx Durable Object 上下文
	 * @param env Worker 环境变量
	 */
	constructor(
		ctx: DurableObjectState,
		env: Env
	) {
		// 必须调用父类构造函数
		super(ctx, env);

		console.log("MyDurableObject 已创建");
	}

	/**
	 * RPC 方法（远程调用方法）
	 *
	 * Worker 可以像调用本地函数一样调用它：
	 *
	 * const result = await stub.sayHello("world")
	 *
	 * Cloudflare 内部会自动：
	 * 1. 找到对应 Durable Object
	 * 2. 转发请求
	 * 3. 执行该方法
	 * 4. 返回结果
	 *
	 * @param name 用户名
	 * @returns 返回问候语
	 */
	async sayHello(name: string): Promise<string> {
		return `Hello, ${name}!`;
	}
}

/**
 * Worker 默认导出
 *
 * 这里相当于整个应用入口
 */
export default {
	/**
	 * fetch 请求入口
	 *
	 * 所有 HTTP 请求都会进入这里
	 *
	 * @param request 用户请求
	 * @param env 环境变量
	 * @param ctx 执行上下文
	 */
	async fetch(
		request: Request,
		env: Env,
		ctx: ExecutionContext
	): Promise<Response> {

		/**
		 * 获取 Durable Object Stub（代理对象）
		 *
		 * stub 可以理解成：
		 * “远程 Durable Object 的客户端”
		 *
		 * getByName("foo") 的含义：
		 * - 获取名字为 foo 的 DO 实例
		 * - 如果不存在则自动创建
		 *
		 * 重点：
		 * 所有使用 "foo" 的请求，
		 * 都会进入同一个 Durable Object 实例
		 */
		const stub = env.MY_DURABLE_OBJECT.getByName("foo");

		/**
		 * 调用 Durable Object 内的方法
		 *
		 * 这里其实是：
		 * Worker -> DO 的远程调用（RPC）
		 */
		const greeting = await stub.sayHello("world");

		/**
		 * 返回 HTTP 响应
		 */
		return new Response(greeting, {
			headers: {
				"content-type": "text/plain;charset=UTF-8",
			},
		});
	},
} satisfies ExportedHandler<Env>;
```

## ctx 是什么

```
ctx: DurableObjectState
```

是 DO 的运行上下文。

里面有：

| API                       | 作用       |
| ------------------------- | ---------- |
| ctx.storage               | 持久化存储 |
| ctx.blockConcurrencyWhile | 锁         |
| ctx.getWebSockets         | websocket  |
| ctx.waitUntil             | 后台任务   |