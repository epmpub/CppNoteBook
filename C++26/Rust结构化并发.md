Rust 在结构化并发（Structured Concurrency）上的支持非常优秀，甚至被认为是现代语言中做得最好的之一。

Rust 的所有权系统 + async/await 天生就倾向于结构化设计，与 C++26 std::execution 相比，Rust 的实现更“隐式”和“编译期强制”。

1. ## Rust 结构化并发的核心机制

- async fn / async block + .await：
  - Future 的生命周期严格绑定到作用域（lexical scope）。
  - co_await（即 .await）是结构化的：子 future 必须在当前 future 完成前完成。
- tokio::join! 和 futures::join!：
  - 同时等待多个 future，全部完成后才继续。
  - 失败时自动传播错误，取消时自动清理。
- tokio::select!：
  - 竞争多个分支，先完成的胜出，其他自动取消。
  - 完美体现“父任务控制子任务”。
- tokio::spawn + JoinHandle / JoinSet：
  - tokio::spawn 默认是 fire-and-forget（非完全结构化）。
  - 推荐使用 JoinSet：把多个任务收集到一个集合中，在作用域结束时自动等待/取消所有子任务，实现真正的结构化。
- std::thread::scope（Rust 1.63+）：
  - 同步线程的结构化并发典范。
  - 子线程生命周期绑定到 scope，scope 结束前必须全部完成，且允许安全借用栈上数据（无需 Arc）。
- rayon::scope：
  - 数据并行（parallel iterators）的结构化支持。
- Rust 结构化并发的优势（与 C++ 对比）

| 特性            | Rust 做法                          | C++26 std::execution      | 优势                                   |
| --------------- | ---------------------------------- | ------------------------- | -------------------------------------- |
| 生命周期绑定    | 所有权 + borrow checker 强制       | Sender/Receiver + 作用域  | 编译期保证，几乎无法写出 dangling task |
| 取消传播        | Drop + Cancellation Token 自动     | 显式 propagation          | 更自动                                 |
| 错误传播        | ? 操作符 + JoinSet                 | Receiver 的 error channel | 更符合 Rust 风格                       |
| Fire-and-Forget | 存在（tokio::spawn）但社区鼓励避免 | 容易出现                  | Rust 更鼓励结构化                      |
| 树状任务结构    | 天然（async 树 + JoinSet）         | 显式构建                  | 更简洁                                 |
| 同步并发        | std::thread::scope                 | 无直接等价                | 非常强大                               |

3. ## 实际推荐写法（Tokio 生态）

rust

```rust
use tokio::{task::JoinSet, select};

async fn structured_example() {
    let mut set = JoinSet::new();
    
    // 结构化 spawn
    for i in 0..10 {
        set.spawn(async move { /* 子任务 */ });
    }
    
    // 等待所有完成或处理取消
    while let Some(res) = set.join_next().await {
        // 处理结果或错误
    }
    
    // scope 结束时自动取消未完成的任务
}
```

使用 CancellationToken 可以更精细控制取消传播。

4. ## Rust 的哲学

- Fearless Concurrency：借用检查器 + Send/Sync 让结构化并发更安全。

- Tree-Structured Concurrency：任务形成清晰的树状层级，父任务负责子任务的生命周期（Yosh Wuyts 等人的理念影响很大）。

- 社区趋势：越来越鼓励使用 JoinSet、select!、join! 而非裸 tokio::spawn + 全局 task。

  

总结：
Rust 在结构化并发上的支持比 C++26 更“开箱即用”和安全，因为借用检查器天然阻止了很多非结构化并发带来的 bug。

std::execution 更像是一个灵活的底层框架，而 Rust 的 async + Tokio 则更接近“结构化优先”的编程体验。

如果你想看具体代码对比（Rust JoinSet vs C++ std::execution::when_all）、Tokio 最佳实践，或与 Python Trio 的对比，随时告诉我！