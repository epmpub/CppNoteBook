C++26 async_scope：为非顺序并发创建作用域（提案 P3149R11，已被纳入 C++26）。这是 std::execution（Sender/Receiver 框架）的重要补充，专注于结构化并发（Structured Concurrency），解决动态数量、fire-and-forget 异步工作的生命周期管理问题。1. 为什么需要 Async Scopes？std::execution（P2300）提供了强大的结构化并发工具（如 when_all、let_value），但在以下场景仍不足：

- 动态数量的并行任务（无法预知 when_all 的子任务数）。
- 从协程或异步函数中“spawn”工作，但需要确保所有工作在作用域结束前完成。
- 逐步将非结构化代码（start_detached）迁移到结构化代码。
- 避免悬垂引用、资源过早销毁、难以调试的 race conditions。

async_scope 像一个异步 nursery（育儿室），它跟踪所有启动的子工作，并在自身销毁前等待它们全部完成 —— 类似 RAII，但用于异步世界。2. 核心组件

- execution::scope_token：令牌，用于将 Sender 与 scope 关联。
- execution::associate / nest（C++26 最终命名）：将 Sender “附加”到 scope。
- execution::spawn：启动工作（fire-and-forget 风格，但被 scope 跟踪）。
- execution::spawn_future：启动工作并返回一个 Sender（可用于获取结果）。
- 两种具体 scope：
  - simple_counting_scope：简单引用计数（无内置取消）。
  - counting_scope：带停止（stop）支持的版本（可 request_stop）。
- 基本用法示例

cpp

```cpp
#include <execution>
namespace ex = std::execution;

ex::counting_scope scope;   // 或 simple_counting_scope

// 在某个异步上下文中
auto work = ex::just(42)
          | ex::then([](int v) { /* ... */ return v * 2; });

// 方式1：spawn（跟踪但不直接获取结果）
ex::spawn(scope, std::move(work));        // 或 scope.spawn(...)

// 方式2：spawn_future（返回可 co_await 或继续的 Sender）
auto future_sender = ex::spawn_future(scope, std::move(work));

// 等待 scope 内所有工作完成
ex::sync_wait(scope.join());
```

与 std::execution::task 配合（C++26）：

cpp

```cpp
ex::task<void> process_requests(ex::counting_scope& scope) {
    while (true) {
        auto req = co_await get_next_request();
        // 动态 spawn 子任务
        ex::spawn(scope, process_one_request(std::move(req)));
    }
    co_await scope.join();   // 确保清理
}
```

4. 关键特性与语义

- 非顺序（Non-sequential）：允许在任意时刻、任意地方启动工作（不像 when_all 必须在同一表达式中列出所有子任务）。
- 结构化保证：scope 析构时（或 join() 完成时）确保所有关联工作已完成 → 资源安全、易于推理。
- 与协程深度集成：可在 ex::task 中安全 spawn。
- 取消支持（counting_scope）：可请求停止，子工作可通过 stop token 响应。
- 组合性：可嵌套、可作为类成员、可与调度器结合。
- 与旧方式对比

| 方式                   | 问题                                | async_scope 的优势             |
| ---------------------- | ----------------------------------- | ------------------------------ |
| start_detached         | 完全 unstructured，生命周期难控     | 结构化跟踪，scope 结束前必完成 |
| when_all + 动态 vector | 笨重，需要手动收集所有 Sender       | 动态 spawn，无需提前收集       |
| 手动 shared_ptr + 计数 | 容易出错、代码冗余                  | 库级支持，RAII-like            |
| 纯协程 co_await        | 适合线性流程，不适合 fan-out/fan-in | 结合使用，完美互补             |

6. 设计哲学

- 无默认 detached work：C++26 鼓励所有异步工作都被某个 scope 管理。
- 渐进式迁移：帮助现有代码从 unstructured 逐步转向 structured。
- 与 std::execution::task 一起构成 C++26 异步编程的核心高层抽象。

总结async_scope（counting_scope 等）填补了 Sender/Receiver 在动态、非线性并发场景下的关键缺失。它让开发者可以安全地“spawn 大量异步工作”，同时保持结构化的生命周期管理，大幅降低悬垂、泄漏和 race 的风险。这是 C++26 结构化并发 拼图的重要一块，与 task、when_all、schedular 等一起，形成了现代、高性能、可组合的异步编程模型。想看具体场景的完整代码示例（HTTP server listener、并行 for 循环、类成员 scope 等）吗？或者与 std::execution::task 混合使用的细节？