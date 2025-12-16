# 1. 安装

```cmd
pnpm add posthog-js @posthog/react
```

```env
VITE_PUBLIC_POSTHOG_KEY=phc_9Y4KAxm8EFxzP3i6y2PhbglAvNMRhEJtjL4f24XxKjW
VITE_PUBLIC_POSTHOG_HOST=https://us.i.posthog.com
```

```tsx
// src/main.jsx
import { StrictMode } from 'react'
import { createRoot } from 'react-dom/client'
import './index.css'
import App from './App.jsx'
import posthog from 'posthog-js';
import { PostHogProvider } from '@posthog/react'

posthog.init(import.meta.env.VITE_PUBLIC_POSTHOG_KEY, {
  api_host: import.meta.env.VITE_PUBLIC_POSTHOG_HOST,
  defaults: '2025-11-30',
});

createRoot(document.getElementById('root')).render(
  <StrictMode>
    <PostHogProvider client={posthog}>
      <App />
    </PostHogProvider>
  </StrictMode>,
)
```

```ts
// ... imports 用了react-router7 时配置vite.config.ts

export default defineConfig({
  plugins: [tailwindcss(), reactRouter(), tsconfigPaths()],
  ssr: {
    noExternal: ['posthog-js', '@posthog/react']
  }
});
```

### 使用 posthog-js 函数

```tsx
import { usePostHog } from '@posthog/react'
import { useEffect } from 'react'
import { useUser, useLogin } from '../lib/user'

function App() {
    const posthog = usePostHog()
    const login = useLogin()
    const user = useUser()

    useEffect(() => {
        if (user) {
            // Identify sends an event, so you may want to limit how often you call it
            // 识别用户为邮箱
            posthog?.identify(user.id, {
                email: user.email,
            })
            // 给用户分组
            posthog?.group('company', user.company_id)
        }
    }, [posthog, user.id, user.email, user.company_id])

    const loginClicked = () => {
        // 用户行为上报
        posthog?.capture('clicked_log_in')
        login()
    }

    return (
        <div className="App">
            {/* Fire a custom event when the button is clicked */}
            <button onClick={() => posthog?.capture('button_clicked')}>Click me</button>
            {/* This button click event is autocaptured by default */}
            <button data-attr="autocapture-button">Autocapture buttons</button>
            {/* This button click event is not autocaptured */}
            <button className="ph-no-capture">Ignore certain elements</button>
            <button onClick={loginClicked}>Login</button>
        </div>
    )
}

export default App
```

### 跟踪元素可见性

```tsx
import { PostHogCaptureOnViewed } from '@posthog/react'

function App() {
    return (
        <PostHogCaptureOnViewed name="hero-banner">
            <div>Your important content here</div>
        </PostHogCaptureOnViewed>
    )
}
```

#### 添加更多属性

```tsx
<PostHogCaptureOnViewed
    name="product-card"
    properties={{ 
        product_id: '123', 
        category: 'electronics',
        price: 299.99
    }}
>
    <ProductCard />
</PostHogCaptureOnViewed>
```

#### 追踪多个子元素

```tsx
<PostHogCaptureOnViewed
    name="product-gallery"
    properties={{ gallery_type: 'featured' }}
    trackAllChildren
>
    <ProductCard id="1" />
    <ProductCard id="2" />
    <ProductCard id="3" />
</PostHogCaptureOnViewed>
启用 trackAllChildren 时，每个子元素会发送带有 child_index 属性的事件，指示其位置。
```

#### 自定义交叉观察者选项

```tsx
<PostHogCaptureOnViewed
    name="footer"
    observerOptions={{ 
        threshold: 0.5,  // Element is 50% visible
        rootMargin: '0px'
    }}
>
    <Footer />
</PostHogCaptureOnViewed>
你可以通过将选项传递给 IntersectionObserver 来自定义元素何时被视为“已查看”：
```

## 2. Feature flags 特征标志

PostHog 的功能[标志](https://posthog.com/docs/feature-flags)使你能够安全地部署和回滚新功能，并针对特定用户和群体进行定位。

在 React 中实现功能标志有两种方式：

1. Using hooks. 用钩子。
2. Using the `<PostHogFeature>` component.
   使用`了 <PostHogFeature>` 组件。

#### 使用钩子函数

`useFeatureFlagEnabled` ` useFeatureFlagVariantKey` ` useActiveFeatureFlags` `useFeatrueFlagePayload`

| Hook 钩                    | Description 描述                                             |
| -------------------------- | ------------------------------------------------------------ |
| `useFeatureFlagEnabled`    | `$feature_flag_called` event. 返回一个布尔值，表示特征标志是否被启用。这会发送一个 `$feature_flag_call` 事件。 |
| `useFeatureFlagVariantKey` | 返回功能标志的变体键。这会发送一个 `$feature_flag_call` 事件。 |
| `useActiveFeatureFlags`    | 一系列活跃的功能标志。这*不会*发送 `$feature_flag_call` 事件。 |
| `useFeatureFlagPayload`    | 返回特征标志的有效载荷。这*不会*发送 `$feature_flag_call` 事件。一定要用 `useFeatureFlagEnabled` 或 `useFeatureFlagVariantKey` 一起使用。 |

#### 示例1：使用布尔特征标志

```tsx
import { useFeatureFlagEnabled } from '@posthog/react'

function App() {
  const showWelcomeMessage = useFeatureFlagEnabled('flag-key')
  const payload = useFeatureFlagPayload('flag-key')
  // 功能是否显示
  return (
    <div className="App">
      {
        showWelcomeMessage ? (
          <div>
            <h1>Welcome!</h1>
            <p>Thanks for trying out our feature flags.</p>
          </div>
        ) : (
          <div>
            <h2>No welcome message</h2>
            <p>Because the feature flag evaluated to false.</p>
          </div>
        )
      }
    </div>
  );
}

export default App;
```

#### 示例2：使用多元特征标志

```tsx
import { useFeatureFlagVariantKey } from '@posthog/react'

function App() {
  const variantKey = useFeatureFlagVariantKey('show-welcome-message') // 获取配置枚举值
  let welcomeMessage = '' 
  if (variantKey === 'variant-a') {
    welcomeMessage = 'Welcome to the Alpha!'
  } else if (variantKey === 'variant-b') {
    welcomeMessage = 'Welcome to the Beta!'
  }  

  return (
    <div className="App">
      {
        welcomeMessage ? (
          <div>
            <h1>{welcomeMessage}</h1>
            <p>Thanks for trying out our feature flags.</p>
          </div>
        ) : (
          <div>
            <h2>No welcome message</h2>
            <p>Because the feature flag evaluated to false.</p>
          </div>
        )
      }
    </div>
  );
}

export default App;
```

#### 示例3：使用标志有效载荷

`useFeatureFlagPayload` *钩子不会*发送 [`$feature_flag_called`](https://posthog.com/docs/experiments/new-experimentation-engine#experiment-exposure) 事件，而该事件是实验跟踪所必需的。为了确保暴露事件被发送，你应**始终**使用 `useFeatureFlagPayload` 钩子，同时使用 `useFeatureFlagEnabled` 或 `useFeatureFlagVariantKey` 钩子。

```tsx
import { useFeatureFlagPayload } from '@posthog/react'

function App() {
  const variant = useFeatureFlagEnabled('show-welcome-message') // 获取配置功能是否开启
  const payload = useFeatureFlagPayload('show-welcome-message') // 获取配置数据

    return (
                <>
                {
                    variant ? (
                        <div className="welcome-message">
                            <h2>{payload?.welcomeTitle}</h2>
                            <p>{payload?.welcomeMessage}</p>
                        </div>
                    ) : <div>
                        <h2>No custom welcome message</h2>
                        <p>Because the feature flag evaluated to false.</p>
                    </div>
                }
        </>
    )
}
```

### 方法二：使用 PostHogFeature 组件

`PostHogFeature` 组件通过处理与特征标志相关的逻辑简化了代码。
它还会自动捕捉指标，比如用户与此功能互动的次数

tips: 需要配置posthogprovider

```tsx
import { PostHogFeature } from '@posthog/react'

function App() {
    return (
        <PostHogFeature flag='show-welcome-message' match={true}>
            <div>
                <h1>Hello</h1>
                <p>Thanks for trying out our feature flags.</p>
            </div>
        </PostHogFeature>
    )
}
```

#### Payloads 

```tsx
import { PostHogFeature } from '@posthog/react'

function App() {

    return (
        <PostHogFeature flag='show-welcome-message' match={true}>
           {(payload) => {
                return (
                    <div>
                        <h1>{payload.welcomeMessage}</h1>
                        <p>Thanks for trying out our feature flags.</p>
                    </div>
                )
           }}
        </PostHogFeature>
    )
}
```

### Request timeout  请求暂停

你可以在初始化 PostHog 客户端时设置 `feature_flag_request_timeout_ms` 参数，设置标志请求超时。这有助于防止 PostHog 服务器响应过慢时你的代码被封锁。默认时长设为 3 秒。

```tsx
posthog.init('phc_9Y4KAxm8EFxzP3i6y2PhbglAvNMRhEJtjL4f24XxKjW', { 
  api_host: 'https://us.i.posthog.com',
  defaults: '2025-11-30'
  feature_flag_request_timeout_ms: 3000 // Time in milliseconds. Default is 3000 (3 seconds).
}
)
```

### Error handling 错误处理

使用 PostHog SDK 时，处理功能标志作中可能出现的错误非常重要。这里有一个如何将 PostHog SDK 方法封装在错误处理程序中的示例：新功能是否使用以及请求是否回滚机制

```tsx
function handleFeatureFlag(client, flagKey, distinctId) {
    try {
        const isEnabled = client.isFeatureEnabled(flagKey, distinctId);
        console.log(`Feature flag '${flagKey}' for user '${distinctId}' is ${isEnabled ? 'enabled' : 'disabled'}`);
        return isEnabled;
    } catch (error) {
        console.error(`Error fetching feature flag '${flagKey}': ${error.message}`);
        // Optionally, you can return a default value or throw the error
        // return false; // Default to disabled
        throw error;
    }
}

// Usage example
try {
    const flagEnabled = handleFeatureFlag(client, 'new-feature', 'user-123');
    if (flagEnabled) {  
        // Implement new feature logic
    } else {
        // Implement old feature logic
    }
} catch (error) {
    // Handle the error at a higher level
    console.error('Feature flag check failed, using default behavior');
    // Implement fallback logic
}
```

注销用户

```js
posthog.reset() // 登出后
```

#### 重置设备id

```js
posthog.reset(true)
```

event identify
