### 权限配置

```js
const headers = {
    Authorization: `Bearer ${POSTHOG_PERSONAL_API_KEY}`
}
```

`or`

```js
const body = {
    personal_api_key: POSTHOG_PERSONAL_API_KEY
}
```

### 速率限制

Query 2400/hour

### 接口

你必须向正确的域名发送 API 请求。在美国云端，这些 `https://us.i.posthog.com` 用于公共端点，`https://us.posthog.com` 用于私有端点。在欧盟云端，这些 `https://eu.i.posthog.com` 用于公共端点，`https://eu.posthog.com` 用于私有端点。对于自托管实例，使用你的自托管域名。通过检查你的 PostHog 实例 URL 来确认你的。

多个应用端 最好放在一个项目中

POST 请求体的最大大小由 `settings.DATA_UPLOAD_MAX_MEMORY_SIZE` 控制，默认为 20MB。

### Capture

每个事件请求必须包含 `api_key`、`distinct_id` 和`event`字段，并带有名称。` properties`和`timestamp`字段都是可选的

```js
import fetch from "node-fetch";

async function sendPosthogEvent() {
  const url = "https://us.i.posthog.com/i/v0/e/";
  const headers = {
      "Content-Type": "application/json",
  };
  const payload = {
    api_key: "phc_9Y4KAxm8EFxzP3i6y2PhbglAvNMRhEJtjL4f24XxKjW",
    event: "event name",
    distinct_id: "user distinct id",
    properties: {
      account_type: "pro",
    },
    timestamp: "[optional timestamp in ISO 8601 format]",
  };

  const response = await fetch(url, {
    method: "POST",
    headers: headers,
    body: JSON.stringify(payload),
  });

  const data = await response.json();
  console.log(data);
}

sendPosthogEvent()
```

#### 匿名事件

```js
import fetch from "node-fetch";

async function sendAnonymousPosthogEvent() {
  const url = "https://us.i.posthog.com/i/v0/e/";
  const headers = {
      "Content-Type": "application/json",
  };
  const payload = {
    api_key: "phc_9Y4KAxm8EFxzP3i6y2PhbglAvNMRhEJtjL4f24XxKjW",
    event: "event name",
    distinct_id: "user distinct id",
    properties: {
      $process_person_profile: false
    }
  };

  const response = await fetch(url, {
    method: "POST",
    headers: headers,
    body: JSON.stringify(payload),
  });

  const data = await response.json();
  console.log(data);
}

sendAnonymousPosthogEvent()
```

#### Batch events

```js
curl -v -L --header "Content-Type: application/json" -d '{
  "api_key": "phc_9Y4KAxm8EFxzP3i6y2PhbglAvNMRhEJtjL4f24XxKjW",
  "historical_migration": false,
  "batch": [
    {
      "event": "batched_event_name_1",
      "properties": {
        "distinct_id": "user distinct id",
        "account_type": "pro"
      },
      "timestamp": "[optional timestamp in ISO 8601 format]"
    },
    {
      "event": "batched_event_name_2",
      "properties": {
        "distinct_id": "user distinct id",
        "account_type": "pro"
      }
    }
  ]
}' https://us.i.posthog.com/batch/
```

#### Historical migrations 历史迁徙

```cmd
curl -v -L --header "Content-Type: application/json" -d '{
  "api_key": "phc_9Y4KAxm8EFxzP3i6y2PhbglAvNMRhEJtjL4f24XxKjW",
  "historical_migration": true,
  "batch": [
    {
      "event": "batched_event_name",
      "properties": {
        "distinct_id": "user_id"
      },
      "timestamp": "2024-04-03T12:00:00Z"
    },
    {
      "event": "batched_event_name",
      "properties": {
        "distinct_id": "user_id"
      },
      "timestamp": "2024-04-03T12:00:00Z"
    }
  ]
}' https://us.i.posthog.com/batch/
```



#### Alias 别名

```cmd
curl -v -L --header "Content-Type: application/json" -d '{
  "api_key": "phc_9Y4KAxm8EFxzP3i6y2PhbglAvNMRhEJtjL4f24XxKjW",
  "event": "$create_alias",
  "distinct_id": "123",
  "properties": {
    "alias": "456"
  }
}' https://us.i.posthog.com/i/v0/e/
```

#### Group identify 团体识别

```cmd
curl -v -L --header "Content-Type: application/json" -d '{
  "api_key": "phc_9Y4KAxm8EFxzP3i6y2PhbglAvNMRhEJtjL4f24XxKjW",
  "event": "$groupidentify",
  "distinct_id": "groups_setup_id",
  "properties": {
    "$group_type": "<group_type>",
    "$group_key": "<company_name>",
    "$group_set": {
      "name": "<company_name>",
      "subscription": "premium"
      "date_joined": "[optional timestamp in ISO 8601 format]"
    }
  }
}' https://us.i.posthog.com/i/v0/e/
```

#### Groups 组

```cmd
curl -v -L --header "Content-Type: application/json" -d '{
  "api_key": "phc_9Y4KAxm8EFxzP3i6y2PhbglAvNMRhEJtjL4f24XxKjW",
  "event": "event name",
  "distinct_id": "user distinct id",
  "properties": {
    "$groups": {"company": "<company_name>"}
  }
}' https://us.i.posthog.com/i/v0/e/
```

#### Identify 识别

```cmd
curl -v -L --header "Content-Type: application/json" -d '{
  "api_key": "phc_9Y4KAxm8EFxzP3i6y2PhbglAvNMRhEJtjL4f24XxKjW",
  "event": "$identify",
  "distinct_id": "user distinct id",
  "properties": {
    "$set": {
      "is_cool": "true"
    }
  },
  "timestamp": "2020-08-16T09:03:11.913767"
}' https://us.i.posthog.com/i/v0/e/
```

**注：**`$identify` 事件的工作方式与 [JavaScript SDK](https://posthog.com/docs/libraries/js/features#identifying-users) 中的 `identify（）` 方法不同。该事件更新了 person 属性，而 JavaScript 的 `identify（）` 方法则连接匿名用户和一个不同的 ID。

#### Pageview 页面浏览

```cmd
curl -v -L --header "Content-Type: application/json" -d '{
  "api_key": "phc_9Y4KAxm8EFxzP3i6y2PhbglAvNMRhEJtjL4f24XxKjW",
  "event": "$pageview",
  "distinct_id": "user distinct id",
  "properties": {
    "$current_url": "/docs/api/capture",
    "$session_id": "019a0ccb-408c-728a-9df9-1ef51b742b36"
  }
}' https://us.i.posthog.com/i/v0/e/
```

默认的 PostHog 事件和属性带有 `$` 前缀。其中最常见且最受欢迎的是 `$pageview` 活动

#### Screen view 屏幕视图

相当于移动应用的页面浏览量。

```cmd
curl -v -L --header "Content-Type: application/json" -d '{
  "api_key": "phc_9Y4KAxm8EFxzP3i6y2PhbglAvNMRhEJtjL4f24XxKjW",
  "event": "$screen",
  "distinct_id": "user distinct id",
  "properties": {
    "$screen_name": "TheScreen"
  }
}' https://us.i.posthog.com/i/v0/e/
```

#### Survey 调查

```cmd
curl -v -L --header "Content-Type: application/json" -d '{
  "api_key": "phc_9Y4KAxm8EFxzP3i6y2PhbglAvNMRhEJtjL4f24XxKjW",
  "event": "survey sent",
  "distinct_id": "user distinct id",
  "properties": {
    "$survey_id": "survey_id",
    "$survey_response_d8462827-1575-4e1e-ab1d-b5fddd9f829c": "Awesome!",
    "$survey_questions": [
      {
        "id": "d8462827-1575-4e1e-ab1d-b5fddd9f829c",
        "question": "How likely are you to recommend us to a friend?"
      }
    ]
  }
}' https://us.i.posthog.com/i/v0/e/
```

