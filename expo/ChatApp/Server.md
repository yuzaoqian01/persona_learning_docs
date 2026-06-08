# chat 服务端设计

准备使用cloud flare 来实现前期chat 服务 因为这些都免费且通用 DO天然支持持久化时序

整体架构

```json
客户端 App
  ├─ SQLite：本地消息、会话、发送队列
  ├─ WebSocket：实时收发消息
  └─ HTTP API：登录、拉历史、上传附件、会话列表

Cloudflare Worker API
  ├─ 鉴权
  ├─ 路由请求
  ├─ 查询 D1/Postgres
  ├─ 生成 R2 上传 URL
  └─ 获取 Conversation DO Stub

Conversation DO
  ├─ 管 WebSocket 连接
  ├─ 生成 seq
  ├─ 消息去重 clientMsgId
  ├─ 写 recent_messages
  ├─ 广播在线用户
  └─ 写长期库 D1/Postgres

D1/Postgres
  ├─ conversations
  ├─ conversation_members
  ├─ messages
  └─ attachments

KV
  ├─ lastMessage 缓存
  ├─ user profile 缓存
  └─ group meta 缓存

R2
  └─ 图片 / 视频 / 语音 / 文件
```

tasks

```js
1. 建 D1/Postgres 表
2. 写 Conversation DO
3. DO 支持 WebSocket connect
4. DO 支持 send_message
5. DO 生成 seq
6. DO 写 recent_messages
7. DO 写 messages 长期表
8. DO 广播消息
9. 客户端 SQLite 存 local_messages
10. 做历史分页
11. 做重连补消息
12. 做会话列表
13. 做已读未读
14. 做 R2 附件
15. 做 KV 缓存优化
```

