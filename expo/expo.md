# 一、模式与工具关系表

| 名称                 | 类型 / 模式        | 核心特点                                                     | 能否改原生                     | 典型用途                                             |
| -------------------- | ------------------ | ------------------------------------------------------------ | ------------------------------ | ---------------------------------------------------- |
| **Expo Go**          | App 客户端         | 已编译好的 Expo App，可扫码运行                              | ❌ 不可                         | Demo / UI 调试 / JS-only 逻辑                        |
| **Managed Workflow** | 项目模式           | 不含 `ios/`、`android/` 原生目录，Expo 管理原生              | ❌ 不可（只能用 config plugin） | 快速开发、业务 App、轻量 SDK                         |
| **Bare Workflow**    | 项目模式           | 有 `ios/`、`android/` 原生目录，自由修改 Podfile / Gradle    | ✅ 可控                         | 需要原生 SDK / 高自由度项目                          |
| **bare-minimum**     | Bare Workflow 模板 | 极简 Bare Workflow，只生成必要目录                           | ✅ 可控                         | 新项目起步、打算加钱包 / MPC / 原生模块              |
| **prebuild**         | 工具               | 将 Managed Workflow 项目自动生成 `ios/`/`android/` 目录 → 转成 Bare Workflow | ✅ 可控（生成后即可修改）       | 需要原生 SDK / 自定义依赖，但想先用 Expo CLI 管理 JS |
| **expo-dev-client**  | 工具 / 客户端      | 自定义 Dev Client，可以运行你的原生模块（不受 Expo Go 限制） | ✅ 可控                         | 连接你的 Metro、调试原生模块、MPC / 钱包开发         |

# 创建expo应用程序



一个用于创建新的 Expo 和 React Native 项目的命令行工具。

------

`create-expo-app`是一个用于创建和设置新的 Expo 和 React Native 项目的命令行工具。该工具提供各种模板，简化了初始化过程，无需手动配置即可快速上手。

## 创建一个新项目

要创建新项目，请运行以下命令：

终端npm纱pnpm包子

```
npx create-expo-app@latest
```

运行上述命令后，系统会提示您输入项目的应用程序名称。此应用程序名称也会用于应用程序配置的[`name`](https://docs.expo.dev/versions/latest/config/app/#name)属性中。

终端

```
What is your app named? my-app
```

## 选项

使用以下选项自定义命令行为。

### `--yes`

使用默认选项创建新项目。

### `--no-install`

跳过安装 npm 依赖项或 CocoaPods。

### `--template`

`create-expo-app`使用[Node 包管理器](https://docs.expo.dev/more/create-expo/#node-package-managers-support)运行会初始化并使用默认模板设置一个新的 Expo 项目。

您可以使用该`--template`选项选择以下模板之一，或将其作为参数传递给该选项。例如，`--template default`.

> 想要更多模板？不妨试试[`--example`](https://docs.expo.dev/more/create-expo/#--example)使用示例应用程序初始化您的项目，这些应用程序演示了特定功能和集成。

| 模板                                                         | 描述                                                         |
| :----------------------------------------------------------- | :----------------------------------------------------------- |
| [`default`](https://github.com/expo/expo/tree/main/templates/expo-template-default) | 默认模板。专为构建多屏应用而设计。包含推荐工具，例如 Expo CLI、Expo Router 库，并已启用 TypeScript 配置。适用于大多数应用。 |
| [`blank`](https://github.com/expo/expo/tree/main/templates/expo-template-blank) | 安装最少所需的 npm 依赖项，而不配置导航。                    |
| [`blank-typescript`](https://github.com/expo/expo/tree/main/templates/expo-template-blank-typescript) | 启用 TypeScript 的空白模板。                                 |
| [`tabs`](https://github.com/expo/expo/tree/main/templates/expo-template-tabs) | 安装并配置基于文件的路由，启用 Expo Router 和 TypeScript。   |
| [`bare-minimum`](https://github.com/expo/expo/tree/main/templates/expo-template-bare-minimum) | 生成包含原生目录（ android和ios ）的空白模板。[`npx expo prebuild`](https://docs.expo.dev/workflow/prebuild/)在安装过程中运行。 |

- 运行此程序`npx create-expo-app --example with-router`会使用 Expo Router 库设置项目。
- 运行此操作`npx create-expo-app --example with-react-navigation`会创建一个类似于默认模板的项目，但配置的是纯 React Navigation 库