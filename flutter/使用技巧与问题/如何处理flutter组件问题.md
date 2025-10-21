## 一. 什么时候用 **方法组件**

 如果满足以下条件，直接用方法组件就好：

- UI **很小**、很简单（例如一个按钮、一段文字组合）。
- **没有状态**（不需要 `setState` / `BlocBuilder` / `Riverpod` 等监听）。
- 只是为了代码拆分，让 `build` 方法更清晰。
- 性能要求高时，function widget 避免了类的额外开销。

例子：

```dart
Widget MyTitle(String text) {
  return Text(
    text,
    style: const TextStyle(fontSize: 20, fontWeight: FontWeight.bold),
  );
}
```

调用：

```dart
Column(
  children: [
    MyTitle("Hello"),
    MyTitle("World"),
  ],
);
```

------

## 二. 什么时候用 **类组件**

 需要更复杂逻辑时，必须用类：

- **有状态** → 用 `StatefulWidget`（计数器、动画、网络请求）。
- **需要生命周期**（`initState` / `dispose` / `didUpdateWidget` 等）。
- **复用性高的 UI 组件**（例如自定义按钮、输入框、卡片）。
- **需要 context 的局部作用域**（比如用 `Theme.of(context)`、`MediaQuery`、`BlocProvider` 等）。

例子：

```dart
class CounterWidget extends StatefulWidget {
  const CounterWidget({super.key});

  @override
  State<CounterWidget> createState() => _CounterWidgetState();
}

class _CounterWidgetState extends State<CounterWidget> {
  int count = 0;

  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        Text("Count: $count"),
        ElevatedButton(
          onPressed: () => setState(() => count++),
          child: const Text("Increment"),
        ),
      ],
    );
  }
}
```

------

## 简单总结

- **方法组件** → 小 UI、无状态、只是提取 UI。
- **类组件** → 有状态、需要生命周期、需要复杂逻辑或封装。

------