C++26 std::execution::parallel_scheduler（提案 P2079R9，已被纳入 C++26）。这是 std::execution（Sender/Receiver 框架）中标准化的并行执行上下文，为开发者提供一个全局、可共享、可替换的并行调度器，解决了之前缺少“默认并行后端”的痛点。1. 为什么需要 parallel_scheduler？

- P2300（std::execution）定义了强大的框架（Scheduler、Sender、Receiver），但没有提供标准的执行上下文。
- 之前开发者常用 static_thread_pool，但容易导致CPU 超订阅（oversubscription），尤其在多库组合的大型应用中。
- 不同组件（甚至不同二进制）使用不同线程池会导致嵌套并行、资源争用等问题。
- 目标：提供一个进程全局共享的并行执行资源，支持 parallel forward progress（并行前向进展），并能与 OS 线程池集成。
- 核心接口

cpp

```cpp
namespace std::execution {

    // 获取标准并行调度器（最常用）
    scheduler auto get_parallel_scheduler();   // 或类似名称，具体以最终标准为准

    // 可能还有其他变体，如带参数的版本（C++26 最终形式）
}
```

- 返回的 scheduler 满足 std::execution::scheduler 概念。
- 支持 schedule() 生成 Sender。
- 特别优化了 bulk() 算法（数据并行常用）。
- 使用示例

cpp

```cpp
#include <execution>
namespace ex = std::execution;

int main() {
    auto sch = ex::get_parallel_scheduler();

    // 基本用法
    auto sender = ex::schedule(sch)
                | ex::then([] { 
                    std::cout << "Running on parallel scheduler\n"; 
                  });

    std::this_thread::sync_wait(std::move(sender));

    // 结构化并发 + bulk（数据并行）
    auto work = ex::just()
              | ex::continues_on(sch)           // 或 on(sch, ...)
              | ex::bulk(10000, [](auto idx) {
                    // 并行处理 10000 个元素
                    process_item(idx);
                });

    std::this_thread::sync_wait(std::move(work));
}
```

与 async_scope / task 结合（C++26 结构化并发）：

cpp

```cpp
ex::counting_scope scope;
auto sch = ex::get_parallel_scheduler();

ex::spawn(scope, ex::on(sch, heavy_computation()));
// ... 动态 spawn 更多工作

ex::sync_wait(scope.join());   // 等待所有工作完成
```

4. 关键设计特性

- 共享与全局：进程内所有组件共享同一个底层线程池，避免超订阅。
- 可替换（Replaceable）：允许用户/平台提供自定义实现（链接时或运行时），便于集成 TBB、oneTBB、Grand Central Dispatch、Windows Thread Pool 等。
- Parallel Forward Progress：保证并行执行语义，适合 CPU 密集型工作。
- OS 集成：优先使用操作系统提供的线程池（如果可用），否则回退到实现提供的线程池。
- 与 bulk 深度集成：高效支持数据并行（bulk 算法可被 scheduler 定制，实现 chunked/unchunked 等策略）。
- 生命周期：设计为全局资源，可在 main() 之外安全使用（有适当保证）。
- 与其他 Scheduler 对比

| Scheduler 类型           | 用途               | 并行性   | 共享性   | 典型场景                   |
| ------------------------ | ------------------ | -------- | -------- | -------------------------- |
| get_parallel_scheduler() | 通用 CPU 并行      | Parallel | 全局共享 | 大多数并行工作（推荐默认） |
| static_thread_pool (旧)  | 本地固定大小线程池 | Parallel | 局部     | 简单测试                   |
| GPU / io_uring Scheduler | 异构计算 / I/O     | 特定     | 特定     | 设备特定工作               |
| run_loop                 | 单线程测试         | 无       | -        | 单元测试                   |

6. 总结parallel_scheduler（通过 get_parallel_scheduler() 获取）是 C++26 std::execution 的“默认并行引擎”。它填补了框架中缺失的标准后端，让开发者可以：

- 无需外部依赖就能写可移植的高性能并行代码；
- 在大型应用中安全组合多个库的并行工作；
- 通过替换机制获得极致性能调优能力。

它与 task、async_scope、when_all、bulk 等一起，构成了 C++26 结构化并行与异步编程的核心基础设施。这是“同一份代码，编译期/运行时一致”和“结构化并发”理念的重要落地。需要更详细的 bulk 使用、替换自定义实现、或与协程 task 混合的例子吗？