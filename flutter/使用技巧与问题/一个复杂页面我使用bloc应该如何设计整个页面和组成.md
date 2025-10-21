# 🔹 BLoC 页面设计思路

1. **拆分层次**
   - **UI 层**：`Widget`，只负责显示，不包含业务逻辑。
   - **State 管理层**：`Bloc / Cubit`，负责业务逻辑、状态切换。
   - **Repository 层**：统一封装 API、数据库、区块链交互。
   - **Model 层**：数据模型，通常配合 `Equatable`。

------

1. **页面结构组织**
   - 一个复杂页面 = 一个 **顶层 BlocBuilder**（管理全局状态）。
   - 内部再拆分成多个小 widget，每个小 widget 可以单独监听状态。
   - 避免整个页面因为一个小状态变化而全局刷新。

------

1. **Bloc 设计模式**
   - **Event / State / Bloc（推荐给复杂页面）**
     - `Event`：用户的动作（点击按钮 / 加载数据）。
     - `State`：UI 状态（加载中 / 加载完成 / 出错）。
     - `Bloc`：负责处理 Event -> 输出新 State。
   - **Cubit（推荐给中等复杂度）**
     - 简化版，只写方法，直接 `emit` 状态。

------

# 🔹 示例：Swap 页面 (复杂 BLoC 设计)

### 目录结构

```dart
/swap
  /bloc
    swap_bloc.dart
    swap_event.dart
    swap_state.dart
  /view
    swap_page.dart
    widgets/
      swap_form.dart
      swap_preview.dart
      swap_button.dart
  /repository
    swap_repository.dart
  /model
    swap_request.dart
    swap_result.dart
```



------

###  State 定义

```dart
part of 'swap_bloc.dart';

abstract class SwapState extends Equatable {
  const SwapState();
  @override
  List<Object?> get props => [];
}

class SwapInitial extends SwapState {}

class SwapLoading extends SwapState {}

class SwapLoaded extends SwapState {
  final SwapResult result;
  const SwapLoaded(this.result);

  @override
  List<Object?> get props => [result];
}

class SwapError extends SwapState {
  final String message;
  const SwapError(this.message);

  @override
  List<Object?> get props => [message];
}
```

------

###  Event 定义

```dart
part of 'swap_bloc.dart';

abstract class SwapEvent extends Equatable {
  const SwapEvent();
  @override
  List<Object?> get props => [];
}

class LoadQuote extends SwapEvent {
  final SwapRequest request;
  const LoadQuote(this.request);
}

class ConfirmSwap extends SwapEvent {
  final SwapRequest request;
  const ConfirmSwap(this.request);
}
```

------

###  Bloc 逻辑

```dart
class SwapBloc extends Bloc<SwapEvent, SwapState> {
  final SwapRepository repository;

  SwapBloc(this.repository) : super(SwapInitial()) {
    on<LoadQuote>(_onLoadQuote);
    on<ConfirmSwap>(_onConfirmSwap);
  }

  Future<void> _onLoadQuote(LoadQuote event, Emitter<SwapState> emit) async {
    emit(SwapLoading());
    try {
      final result = await repository.getQuote(event.request);
      emit(SwapLoaded(result));
    } catch (e) {
      emit(SwapError(e.toString()));
    }
  }

  Future<void> _onConfirmSwap(ConfirmSwap event, Emitter<SwapState> emit) async {
    emit(SwapLoading());
    try {
      final txHash = await repository.executeSwap(event.request);
      emit(SwapLoaded(SwapResult(txHash: txHash)));
    } catch (e) {
      emit(SwapError(e.toString()));
    }
  }
}
```

------

###  Repository

```dart
class SwapRepository {
  final LifiService api;

  SwapRepository(this.api);

  Future<SwapResult> getQuote(SwapRequest request) async {
    final response = await api.getQuote(request);
    return SwapResult.fromJson(response);
  }

  Future<String> executeSwap(SwapRequest request) async {
    final response = await api.execute(request);
    return response['txHash'];
  }
}
```

------

###  页面 UI

```dart
class SwapPage extends StatelessWidget {
  const SwapPage({super.key});

  @override
  Widget build(BuildContext context) {
    return BlocProvider(
      create: (_) => SwapBloc(SwapRepository(LifiService())),
      child: Scaffold(
        appBar: AppBar(title: const Text("Swap")),
        body: BlocConsumer<SwapBloc, SwapState>(
          listener: (context, state) {
            if (state is SwapError) {
              ScaffoldMessenger.of(context).showSnackBar(
                SnackBar(content: Text(state.message)),
              );
            }
          },
          builder: (context, state) {
            if (state is SwapInitial) {
              return const SwapForm();
            } else if (state is SwapLoading) {
              return const Center(child: CircularProgressIndicator());
            } else if (state is SwapLoaded) {
              return SwapPreview(result: state.result);
            } else if (state is SwapError) {
              return Center(child: Text("Error: ${state.message}"));
            }
            return const SizedBox.shrink();
          },
        ),
      ),
    );
  }
}
```

------

### 小组件（解耦 UI）

```dart
class SwapForm extends StatelessWidget {
  const SwapForm({super.key});

  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        const Text("输入 Swap 参数"),
        ElevatedButton(
          onPressed: () {
            context.read<SwapBloc>().add(
              LoadQuote(SwapRequest(fromToken: "ETH", toToken: "USDC")),
            );
          },
          child: const Text("获取报价"),
        ),
      ],
    );
  }
}
```

------

# 🔹 总结

1. **Bloc 管理状态**：一个复杂页面最好有一个 Bloc（大状态机）。
2. **UI 解耦**：页面内部用小 Widget，分别消费 state。
3. **Repository 层隔离 API / SDK**，UI 不直接调用接口。
4. **Equatable**：建议在 `State` / `Event` / `Model` 里都用，减少重建。



### 我现在的闪兑页面上部分是选择兑换的操作 下部分是兑换操作的结果与实施兑换的配置 我应该如何设计状态和结构



你这个页面本质上是一个 **上下分区的复杂页面**，上半部分是「用户输入 & 配置」，下半部分是「结果展示 & 确认执行」。
 这种场景下，用 **Bloc 状态分层设计** 特别合适。

------

## 🔹 页面逻辑分区

```
SwapPage
│
├── TopSection (输入配置区)
│   ├── 选择 fromToken、toToken
│   ├── 输入兑换数量
│   └── 点击“获取报价”
│
└── BottomSection (结果 & 执行区)
    ├── 显示报价信息（汇率、gas、price impact）
    ├── 提示用户授权
    └── 确认按钮（执行交易）
```

------

## 🔹 Bloc 事件（Events）

```
abstract class SwapEvent {}

class SwapInputChanged extends SwapEvent {
  final String fromToken;
  final String toToken;
  final String amount;
  SwapInputChanged(this.fromToken, this.toToken, this.amount);
}

class SwapQuoteRequested extends SwapEvent {}   // 用户点击“获取报价”
class SwapConfirmed extends SwapEvent {}        // 用户点击“确认兑换”
class SwapApprovalRequested extends SwapEvent {}// 授权
```

------

## 🔹 Bloc 状态（States）

```
abstract class SwapState {}

class SwapInitial extends SwapState {}   // 初始状态

class SwapInputState extends SwapState { // 上半部分的输入状态
  final String fromToken;
  final String toToken;
  final String amount;
  SwapInputState({
    required this.fromToken,
    required this.toToken,
    required this.amount,
  });
}

class SwapQuoteLoading extends SwapState {}    // 正在获取报价
class SwapQuoteLoaded extends SwapState {      // 报价结果
  final SwapQuote quote;
  SwapQuoteLoaded(this.quote);
}

class SwapApprovalNeeded extends SwapState {   // 需要授权
  final String token;
  SwapApprovalNeeded(this.token);
}

class SwapExecuting extends SwapState {}       // 正在执行兑换
class SwapSuccess extends SwapState {          // 成功
  final String txHash;
  SwapSuccess(this.txHash);
}

class SwapFailure extends SwapState {          // 失败
  final String error;
  SwapFailure(this.error);
}
```

------

## 🔹 UI 绑定（上部分输入、下部分结果）

```
BlocBuilder<SwapBloc, SwapState>(
  builder: (context, state) {
    return Column(
      children: [
        // 上半部分：输入框 & token 选择
        TopSection(
          onInputChanged: (from, to, amount) {
            context.read<SwapBloc>().add(
              SwapInputChanged(from, to, amount),
            );
          },
          onQuoteRequest: () {
            context.read<SwapBloc>().add(SwapQuoteRequested());
          },
        ),

        // 下半部分：根据状态展示
        if (state is SwapQuoteLoading) CircularProgressIndicator(),
        if (state is SwapQuoteLoaded)
          BottomSection(
            quote: state.quote,
            onConfirm: () {
              context.read<SwapBloc>().add(SwapConfirmed());
            },
          ),
        if (state is SwapApprovalNeeded)
          ApprovalSection(
            token: state.token,
            onApprove: () {
              context.read<SwapBloc>().add(SwapApprovalRequested());
            },
          ),
        if (state is SwapExecuting) Text("交易进行中..."),
        if (state is SwapSuccess) Text("✅ 成功: ${state.txHash}"),
        if (state is SwapFailure) Text("❌ 失败: ${state.error}"),
      ],
    );
  },
)
```

------

## 🔹 总结设计思路

- **输入逻辑（上半部分）** → 独立状态 `SwapInputState` 管理（token、数量）
- **结果逻辑（下半部分）** → 状态机驱动（Quote → Approval → Executing → Success/Failure）
- **Bloc 控制状态流转**，UI 只展示对应状态，职责清晰

