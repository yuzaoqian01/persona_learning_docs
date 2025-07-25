# flutter_bloc

 `flutter_bloc` 是一个 Flutter 状态管理库，基于 `Bloc`（Business Logic Component）模式，它帮助你更好地组织代码，使 UI 与业务逻辑解耦，非常适合中大型应用。

```yaml
dependencies:
  flutter_bloc: ^8.1.3  # 或最新版本

```

# Bloc 模式核心概念

| 概念             | 描述                              |
| ---------------- | --------------------------------- |
| **Event**        | 用户行为或系统事件，例如点击按钮  |
| **State**        | 当前 UI 状态                      |
| **Bloc**         | 接收事件，产生新状态              |
| **BlocProvider** | 用于提供 Bloc 给子组件使用        |
| **BlocBuilder**  | 监听状态变化并重建 UI             |
| **BlocListener** | 响应状态变化但不重建 UI（如弹窗） |

## 基本示例：计数器

### 1. 定义事件

```dart
abstract class CounterEvent {}

class Increment extends CounterEvent {}
class Decrement extends CounterEvent {}
```

------

### 2. 定义状态

```dart
class CounterState {
  final int value;

  CounterState(this.value);
}

```

------

### 3. 创建 Bloc

```dart
class CounterBloc extends Bloc<CounterEvent, CounterState> {
  CounterBloc() : super(CounterState(0)) {
    on<Increment>((event, emit) => emit(CounterState(state.value + 1)));
    on<Decrement>((event, emit) => emit(CounterState(state.value - 1)));
  }
}
```

------

### 4. 使用 Bloc

```dart
void main() {
  runApp(
    BlocProvider(
      create: (_) => CounterBloc(),
      child: const MyApp(),
    ),
  );
}
```

------

### 5. 构建 UI

```dart
class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      home: Scaffold(
        appBar: AppBar(title: const Text('flutter_bloc 示例')),
        body: BlocBuilder<CounterBloc, CounterState>(
          builder: (context, state) {
            return Center(child: Text('计数: ${state.value}', style: const TextStyle(fontSize: 24)));
          },
        ),
        floatingActionButton: Row(
          mainAxisAlignment: MainAxisAlignment.end,
          children: [
            FloatingActionButton(
              heroTag: 'inc',
              onPressed: () => context.read<CounterBloc>().add(Increment()),
              child: const Icon(Icons.add),
            ),
            const SizedBox(width: 16),
            FloatingActionButton(
              heroTag: 'dec',
              onPressed: () => context.read<CounterBloc>().add(Decrement()),
              child: const Icon(Icons.remove),
            ),
          ],
        ),
      ),
    );
  }
}
```

------

## 🎯 适用场景

- 多页面状态共享
- 网络请求状态管理（加载中、成功、失败）
- 表单验证
- 动态导航或权限控制
- 多链钱包 DApp 状态管理（你的应用场景）

------

## 📚 推荐结构（feature-first）

```css
lib/
├── main.dart
├── features/
│   └── counter/
│       ├── bloc/
│       │   ├── counter_bloc.dart
│       │   ├── counter_event.dart
│       │   └── counter_state.dart
│       └── view/
│           └── counter_page.dart
```

# 1. Cubit基本使用

```dart
import 'package:flutter_bloc/flutter_bloc.dart';

class CounterCubit extends Cubit<int> {
  CounterCubit() : super(0);

  void increment() => emit(state + 1);
  void decrement() => emit(state - 1);
}

```

# 2. 使用

```dart
void main() {
  runApp(
    BlocProvider(
      create: (_) => CounterCubit(),
      child: const MyApp(),
    ),
  );
}

```

# 3. 构建UI

```dart
class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      home: Scaffold(
        appBar: AppBar(title: const Text("Cubit 示例")),
        body: BlocBuilder<CounterCubit, int>(
          builder: (context, count) {
            return Center(child: Text('计数: $count', style: const TextStyle(fontSize: 24)));
          },
        ),
        floatingActionButton: Row(
          mainAxisAlignment: MainAxisAlignment.end,
          children: [
            FloatingActionButton(
              heroTag: 'inc',
              onPressed: () => context.read<CounterCubit>().increment(),
              child: const Icon(Icons.add),
            ),
            const SizedBox(width: 16),
            FloatingActionButton(
              heroTag: 'dec',
              onPressed: () => context.read<CounterCubit>().decrement(),
              child: const Icon(Icons.remove),
            ),
          ],
        ),
      ),
    );
  }
}

```

# 4. 组织建议（例如 Feature-based）

```css

lib/
├── main.dart
├── features/
│   └── wallet/
│       ├── cubit/
│       │   └── wallet_cubit.dart
│       └── view/
│           └── wallet_page.dart
```

# 同时提供两个 Cubit

比如你有两个状态管理器：

- `CounterCubit`：用于计数
- `AuthCubit`：用于登录状态

### 1. 创建 Cubit 类

```dart
class CounterCubit extends Cubit<int> {
  CounterCubit() : super(0);
  void increment() => emit(state + 1);
}

class AuthCubit extends Cubit<bool> {
  AuthCubit() : super(false);
  void login() => emit(true);
  void logout() => emit(false);
}
```

------

### 2. 使用 `MultiBlocProvider`

```dart
void main() {
  runApp(
    MultiBlocProvider(
      providers: [
        BlocProvider<CounterCubit>(create: (_) => CounterCubit()),
        BlocProvider<AuthCubit>(create: (_) => AuthCubit()),
      ],
      child: const MyApp(),
    ),
  );
}
```

------

### 3. 在页面中使用

```dart
class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      home: Scaffold(
        appBar: AppBar(title: const Text("MultiBlocProvider 示例")),
        body: Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: [
            BlocBuilder<CounterCubit, int>(
              builder: (context, count) => Text("计数: $count", style: const TextStyle(fontSize: 24)),
            ),
            const SizedBox(height: 20),
            BlocBuilder<AuthCubit, bool>(
              builder: (context, isLoggedIn) =>
                  Text("登录状态: ${isLoggedIn ? "已登录" : "未登录"}"),
            ),
          ],
        ),
        floatingActionButton: Column(
          mainAxisAlignment: MainAxisAlignment.end,
          children: [
            FloatingActionButton(
              heroTag: "inc",
              onPressed: () => context.read<CounterCubit>().increment(),
              child: const Icon(Icons.add),
            ),
            const SizedBox(height: 10),
            FloatingActionButton(
              heroTag: "login",
              onPressed: () {
                final auth = context.read<AuthCubit>();
                auth.state ? auth.logout() : auth.login();
              },
              child: const Icon(Icons.login),
            ),
          ],
        ),
      ),
    );
  }
}
```

------

## 🔁 使用 Bloc + Cubit 混合

你可以混合使用多种 Bloc/Cubit 类型：

```dart
MultiBlocProvider(
  providers: [
    BlocProvider(create: (_) => MyBloc()),
    BlocProvider(create: (_) => MyCubit()),
    BlocProvider(create: (_) => AnotherBloc()),
  ],
  child: MyApp(),
)
```

------

## ✅ 小结

| 方式                | 推荐用于                  |
| ------------------- | ------------------------- |
| `BlocProvider`      | 单一 Bloc / Cubit 注入    |
| `MultiBlocProvider` | 同时注入多个 Bloc / Cubit |