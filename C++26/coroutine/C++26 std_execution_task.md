C++26 std::execution::task（Coroutine Task Type）

（提案 P3552R3，已在 Sofia 会议通过，纳入 C++26）。

这是 C++26 中为 std::execution（Sender/Receiver 框架）提供的一个标准协程返回类型，填补了 C++20 协程长期缺少“开箱即用 Task 类型”的空白。 

open-std.org

1. 为什么需要标准 Task 类型？C++20 只提供了协程底层机制（co_await、co_yield、co_return + promise_type），没有标准的高层 Task 类型。开发者必须自己写 struct Task 或依赖第三方库（如 cppcoro、libunifex、stdexec）。C++26 引入 std::execution 后，需要一个与 Sender/Receiver 深度集成的协程类型，让开发者能自然地用 co_await 操作异步 Sender，同时享受结构化并发、取消、调度器亲和等特性。2. 基本用法

cpp

```cpp
#include <execution>
#include <iostream>
#include <task>          // 或直接在 <execution> 中（具体头文件以最终标准为准）

namespace ex = std::execution;

ex::task<int> compute() {
    std::cout << "Starting computation...\n";
    co_await ex::just(42);           // co_await Sender
    // co_await ex::schedule(some_scheduler); // 切换调度器
    co_return 100;
}

ex::task<void> main_task() {
    int result = co_await compute();
    std::cout << "Result: " << result << '\n';
    co_return;
}

int main() {
    ex::sync_wait(main_task());   // 阻塞等待顶层 task 完成
}
```

关键特性：

- 返回 ex::task<T> 的函数自动成为协程。
- co_await 可以直接用于 Sender（通过 with_awaitable_senders 和 as_awaitable）。
- 支持值返回（T）、void 和异常传播。
- 调度器亲和（scheduler-affine）：协程通常在启动时的调度器上恢复（除非显式切换）。
- 核心设计

- 头文件：<execution>（或配套 <task>）。
- 命名空间：std::execution::task<T>（常简写为 ex::task<T>）。
- 与 Sender/Receiver 集成：
  - task 本身也是 Sender。
  - 支持 set_value、set_error、set_stopped。
  - 取消通过自定义机制（如 unhandled_stopped）处理。
- Promise 类型：使用 with_awaitable_senders 作为基类，让 Sender 在协程中可被 co_await。
- 启动行为：通常延迟启动（lazy），直到被 co_await 或显式启动。
- 与之前 Task 实现的对比

| 特性                           | 第三方库 (cppcoro 等) | std::execution::task (C++26)  |
| ------------------------------ | --------------------- | ----------------------------- |
| Sender 集成                    | 部分支持              | 原生深度集成                  |
| 调度器亲和                     | 可选                  | 默认支持                      |
| 结构化并发                     | 依赖库                | 与 when_all、let_value 等无缝 |
| 标准性                         | 非标准                | 标准库                        |
| 取消支持                       | 各不相同              | 与 Sender 取消通道对齐        |
| 对称转移（Symmetric Transfer） | 通常支持              | 当前版本可能不足（有争议）    |

5. 注意事项与争议（P3801R0）虽然已被采纳，但设计仍有争议：

- 栈溢出风险：迭代式 co_await 在某些调度器（如 inline_scheduler）下可能因缺少对称转移而积累栈帧。
- 参数生命周期：协程参数的析构时机与普通函数不同（可能在整个异步流水线结束后才销毁）。
- 悬垂引用：引用参数仍需小心（与所有协程相同）。
- 语法小问题：如用 co_yield with_error(...) 处理非异常错误。

这些问题可能在 C++26 最终定稿或后续版本中通过 NB 评论或缺陷报告修复。 

open-std.org

6. 配套工具

- ex::sync_wait / this_thread::sync_wait：阻塞等待 task 完成。
- ex::when_all、ex::then、ex::continues_on 等 Sender 算法。
- ex::schedule、ex::starts_on 等调度。
- with_awaitable_senders：自定义协程 promise 的基类。

总结std::execution::task 是 C++26 中协程 + 异步编程的“胶水”，让开发者终于可以用接近现代语言（如 C# async Task、Rust async fn）的风格编写异步代码，同时完全兼容强大的 Sender/Receiver 框架。它不是“最完美”的设计（时间紧迫），但提供了急需的标准词汇类型，极大降低协程入门门槛，并为生态（如网络、并行计算、游戏、嵌入式）打下基础。想看更详细的例子（when_all + task、错误处理、调度器切换、自定义 promise）吗？