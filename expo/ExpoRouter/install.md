### 入口

```json
{
  "main": "expo-router/entry"
}

```

### 配置 babel.config.js

```js
module.exports = function (api) {
  api.cache(true);
  return {
    presets: ['babel-preset-expo'],
    plugins: ['expo-router/babel'],
  };
};

```

路由解析规则

- **app/index.js** 匹配 `/`
- **app/home.js** 匹配 `/home`
- **app/settings/index.js** 匹配 `/settings`
- **app/[user].js** 匹配 `/expo` 或 `/evanbacon` 等动态路径

### Router对象方法

`router` 对象是不可变的，包含以下函数：

- **push**： `(href: Href) => void` 导航至路由。 你可以提供完整路径（如 **/profile/settings**）或相对路径（如 **../settings**）。 通过传递 `{ pathname: 'profile', params: { id: '123' } }` 等对象导航到动态路由。
- **replace**： `(href: Href) => void` 与推送相同的 API，但替换历史记录中的当前路由，而不是推送新路由。 这对于重定向很有用。
- **back**： `() => void` 导航回之前的路由。
- **canGoBack**： `() => boolean` 如果存在有效的历史堆栈并且 `back()` 函数可以弹出，则返回 `true`。
- **setParams**： `(params: Record<string, string>) => void` 更新当前所选路由的查询参数。

### Link 的使用

```jsx

import { Link } from 'expo-router';

// 动态路由
export default function Page() {
  return (
    <View>
      <Link
        href={{
          pathname: "/user/[id]",
          params: { id: 'bacon' }
        }}>
          View user
        </Link>
    </View>
  );
}

// 更换屏幕
export default function Page() {
  return (
    <View>
      <Link
        replace
        href="/feed">
        Login
      </Link>
    </View>
  );
}

```

原生导航并不总是支持 `replace`。 例如，在 Twitter 上，你无法直接从个人资料 "replace" 到推文，这是因为 UI 需要后退按钮才能返回提要或其他顶层选项卡屏幕。 在这种情况下，替换将切换到提要选项卡，并将推文路由推到其顶部，或者如果你在提要选项卡内的其他推文上，它将用新推文替换当前推文。 通过使用 [`unstable_settings`](https://expo.nodejs.cn/router/advanced/router-settings/)，可以在 Expo Router 中获得这种确切的行为