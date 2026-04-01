### 命令

初始化

```cmd
supabase init
```

开启本地环境

```cmd
supabase start

-x, --exclude <strings>

启动可忽略服务
不启动的容器名称。[gotrue,realtime,storage-api,imgproxy,kong,mailpit,postgrest,postgres-meta,studio,edge-runtime,logflare,vector,supavisor]

管理数据库迁移- supabase migration
用于发布到生产环境的 CI/CD supabase db push
管理您的 Supabase 项目：supabase projects
直接从数据库模式生成类型：supabase gen types
Shell 自动补全：supabase completion
```

​	停止

```cmd
supabase stop
```

关联远程项目

```cmd
supabase link --project-ref ********************
```

```cmd
supabase functions deploy ${functionName}
```

```cmd
查看本地 migration
supabase migration list
.本地所有 migration 文件
.已应用 / 未应用状态
```



```cmd
查看远程数据库 migration 状态
supabase db remote commit
```

```cmd
生成新的迁移
supabase migration new xxx
```

```cmd
数据迁移推送
supabase db push
```

