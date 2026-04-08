Process.nextTick

`process.nextTick()` 会把回调放到 **当前执行栈结束后立刻执行**，优先级高于普通的异步任务（比如 `setTimeout`、`setImmediate`）

定时器中采用红黑树的操作时间复杂度为0(lg(n)) ,nextTick()为0(1)