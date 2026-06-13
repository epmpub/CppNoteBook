`C++26 std::execution`（P2300，通常称为 **Senders/Receivers**）可以看作：

> C++ 标准库第一次正式把“异步执行模型”提升为一等公民。

它并不是简单的 coroutine 扩展，而是：

- coroutine
- thread pool
- async IO
- scheduler
- pipeline
- cancellation
- structured async

这些东西的统一抽象。

而很多能力，其实 Rust 生态（Tokio / async-await / Future）已经比较成熟。



C++26 `std::execution` 主要是在补：

| Rust 已有能力               | C++ 以前缺失的问题                   |
| --------------------------- | ------------------------------------ |
| async runtime               | C++ 没统一 runtime                   |
| Future polling model        | C++ coroutine 只有语法，没有执行模型 |
| async composition           | C++ async 很难组合                   |
| structured concurrency      | C++ 缺统一任务树                     |
| cancellation token          | C++ cancellation 很碎片化            |
| scheduler abstraction       | C++ scheduler 五花八门               |
| async pipeline              | C++ continuation 地狱                |
| executor/runtime separation | C++ executor 标准化失败多年          |

P2300 本质上是：

> 给 C++ coroutine 补“操作系统级”的异步生态。

------

# 1. C++20 coroutine 到底缺了什么？

很多人误以为：

> C++20 coroutine = Rust async/await

其实只完成了：

- suspend/resume 语法
- 状态机生成
- co_await 机制

仅仅是：

> “编译器级 coroutine lowering”

并没有：

- scheduler
- runtime
- event loop
- task model
- cancellation
- structured async
- async combinator

所以：

```cpp
co_await xxx;
```

到底：

- 谁恢复？
- 在哪个线程恢复？
- 怎样取消？
- 怎么组合多个 task？
- 怎么超时？
- 怎么切 scheduler？

标准库完全没定义。

这就是：

> “只有 coroutine syntax，没有 async model”

------

# 2. Rust 已经具备的完整 async 模型

Rust：

```rust
async fn foo() {}
```

背后其实包含：

------

## (1) Future trait

核心：

```rust
trait Future {
    fn poll(...)
}
```

这定义了：

- 挂起
- 恢复
- readiness
- lazy execution

C++20 coroutine 没有统一 poll model。

------

## (2) Runtime

Rust 有：

- Tokio
- async-std
- smol

提供：

- IO reactor
- scheduler
- timers
- cancellation
- task spawning

而 C++：

- Asio
- Folly
- libunifex
- cppcoro
- HPX

全部碎片化。

------

## (3) Structured async

Rust：

```rust
tokio::join!
tokio::select!
```

天然支持：

- fan-out
- fan-in
- racing
- cancellation propagation

而 C++20：

几乎全靠手写。

------

## 3. std::execution（P2300）补了什么？

核心思想：

------

“把 async operation 变成可组合的数据流”

即：

```cpp
sender
```

表示：

> “未来会产生结果的 computation”

类似：

Rust Future。

------

## 4. sender ≈ Rust Future

Rust：

```rust
Future<Output=T>
```

C++26：

```cpp
sender<T>
```

但 sender 更泛化：

它还能表达：

- value
- error
- stopped(cancelled)

三通道模型。

------

## 5. receiver ≈ continuation

receiver：

```cpp
set_value()
set_error()
set_stopped()
```

相当于：

Rust Future poll 完成后的 continuation。

这解决：

> continuation 如何统一表达

------

# 6. scheduler abstraction（重大补齐）

这是 C++ 长期缺失的。

Rust runtime：

```rust
tokio::spawn()
```

默认绑定 Tokio scheduler。

------

C++26：

```cpp
schedule(scheduler)
```

scheduler 成为标准概念。

意味着：

- thread pool
- io_uring
- GPU queue
- NUMA scheduler

都能统一接入。

这是巨大的架构升级。

------

## 7. async pipeline（最核心）

以前 C++ async：

```cpp
future.then(...).then(...)
```

非常碎。

P2300：

```cpp
read_file()
| then(parse)
| then(compute)
| then(write)
```

形成：

> 声明式 async pipeline

这非常像：

Rust：

```rust
stream.map().filter()
```

或者：

```rust
async pipeline
```

------

# 8. completion channels（Rust 没那么显式）

P2300 有三个 completion signal：

```cpp
set_value
set_error
set_stopped
```

这比 Rust Future 更强。

Rust cancellation：

通常：

```rust
Result<T,E>
```

或者 runtime 特殊机制。

而 P2300：

把 cancellation 提升为：

> 一等 completion state。

这是非常现代的设计。

------

## 9. cancellation propagation

以前 C++：

- stop_token
- atomic flag
- future cancel

全是碎片。

P2300：

取消成为 execution graph 的组成部分。

类似：

Rust Tokio task tree cancellation。

------

## 10. Structured Concurrency（补齐重大缺口）

这是现代 async 最重要的东西之一。

Rust Tokio：

```rust
join!
select!
```

任务生命周期形成树。

------

以前 C++：

```cpp
std::thread.detach()
```

灾难级设计。

因为：

- orphan task
- 生命周期失控
- resource leak

------

P2300：

开始进入：

> structured async world

即：

- 父任务管理子任务
- cancellation 自动传播
- scope-based lifetime

这其实是：

Rust async 思想正式进入 C++ 标准。

------

## 11. Executor 失败 → execution 成功

C++ 曾经试图标准化：

```cpp
executor
```

但失败多年。

原因：

不同 runtime 模型差异巨大：

- CPU thread pool
- GPU
- networking
- SIMD
- bulk execution

无法统一。

------

P2300 改变思路：

不标准化 executor API。

而标准化：

## “异步 computation graph”

这是重大思想转变。

------

## 12. Rust 和 C++26 的根本差异

虽然越来越像。

但本质不同：

------

## Rust async

核心：

> runtime-driven

Future 必须 poll。

本质：

```text
runtime owns execution
```

------

## C++26 execution

核心：

> generic execution graph

更强调：

- composability
- scheduler abstraction
- heterogeneous execution

甚至：

- GPU
- SIMD
- HPC

这是 C++ 的传统优势。

------

## 13. C++26 甚至比 Rust 更泛化

Rust async 主要针对：

- networking
- server async IO

------

P2300 目标：

统一：

- CPU task graph
- GPU execution
- sender pipeline
- vectorized execution
- bulk execution
- NUMA execution

它其实更接近：

- Intel TBB flow graph
- HPX
- SYCL
- CUDA graph

------

## 14. 为什么说 P2300 是 C++ 近十年最重要标准之一

因为它第一次真正统一了：

| 能力              | 之前       |
| ----------------- | ---------- |
| coroutine         | 只有语法   |
| async             | 碎片化     |
| executor          | 标准化失败 |
| scheduler         | 无统一     |
| continuation      | 无统一     |
| cancellation      | 混乱       |
| async composition | 极弱       |

P2300：

第一次把：

> “现代异步系统编程模型”

正式放进 ISO C++。

------

## 15. 最终对比总结

| 能力                     | Rust async         | C++20 | C++26 execution |
| ------------------------ | ------------------ | ----- | --------------- |
| coroutine syntax         | ✅                  | ✅     | ✅               |
| runtime model            | ✅                  | ❌     | ✅               |
| scheduler abstraction    | ⚠ runtime-specific | ❌     | ✅               |
| async composition        | ✅                  | ❌     | ✅               |
| structured concurrency   | ✅                  | ❌     | ✅               |
| cancellation propagation | ✅                  | ❌     | ✅               |
| execution graph          | ⚠ 部分             | ❌     | ✅               |
| heterogeneous execution  | ❌                  | ❌     | ✅ 强项          |
| GPU/NUMA integration     | 弱                 | 弱    | 强              |
| async pipeline           | ✅                  | 弱    | ✅               |

------

## 可以这样理解

## Rust async：

更像：

> “现代服务器 async runtime”

------

C++26 execution：

更像：

> “统一整个异构计算世界的 execution framework”

包括：

- async IO
- HPC
- GPU
- SIMD
- pipelines
- task graph
- bulk execution

这是它真正野心所在。