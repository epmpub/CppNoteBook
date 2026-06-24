#### C++26 原子归约操作（Atomic Reduction Operations）

（提案 P3111R8，已被纳入 C++26）。这是 C++26 在原子操作领域另一个重要扩展（与之前的 fetch_min/fetch_max 互补），引入了不返回旧值的“纯归约”原子操作，主要用于高性能计算（HPC）、GPU、并行归约等场景。1. 核心动机传统 fetch_add、fetch_or 等操作是 Read-Modify-Write (RMW)，会返回旧值并参与 acquire 同步，这在以下场景带来问题：

- 性能：很多情况下我们不需要旧值，却被迫承受读取开销（尤其在 GPU / 远端原子操作中）。
- 向量化 / par_unseq：读取操作在 unsequenced 执行策略下可能导致 UB（数据竞争或死锁）。
- 浮点数：fetch_add 要求严格顺序（因为浮点加法不可结合），无法利用树形归约（tree reduction）优化。

新操作解决了这些问题。2. 新增接口（store_* 系列）对 std::atomic<T> 和 std::atomic_ref<T>（适用于整数、指针、浮点类型）新增以下成员函数（返回 void）：

- store_add(T val, memory_order = seq_cst) noexcept
- store_sub(T val, memory_order = seq_cst) noexcept
- store_and(T val, memory_order = seq_cst) noexcept
- store_or(T val, memory_order = seq_cst) noexcept
- store_xor(T val, memory_order = seq_cst) noexcept
- store_min(T val, memory_order = seq_cst) noexcept（与 P3008 浮点 min/max 整合）
- store_max(T val, memory_order = seq_cst) noexcept

也有对应的自由函数：std::atomic_store_add 等。示例：

cpp

```cpp
#include <atomic>
#include <execution>
#include <vector>

std::atomic<int> counter{0};
std::vector<int> data = ...;

std::for_each(std::execution::par_unseq, data.begin(), data.end(), 
    [&](int x) {
        counter.store_add(x, std::memory_order_relaxed);  // C++26 新操作
        // counter.fetch_add(x, std::memory_order_relaxed); // 旧方式（较慢 + 在 unseq 下 UB）
    });
```

3. 关键语义区别

| 特性           | fetch_add 等（旧）         | store_add 等（新，C++26）                             |
| -------------- | -------------------------- | ----------------------------------------------------- |
| 返回值         | 旧值                       | void（无返回）                                        |
| 是否为“读”操作 | 是（参与 acquire 同步）    | 不是（不参与 acquire 序列）                           |
| par_unseq 支持 | UB（vectorization-unsafe） | 安全（lock-free 时）                                  |
| 浮点数结合性   | 严格顺序（不可树形归约）   | 允许结合（支持 tree reduction）                       |
| 硬件映射       | 常规 RMW 指令              | 专用 reduction 指令（PTX red、ARM STADD、x86 RAO 等） |
| 性能           | 较高延迟                   | 通常更低延迟（可达 1.2x+ 提升）                       |

- 内存序：支持 relaxed 和 release，但因为不是读，不建立 acquire 同步。
- 浮点数：默认允许“广义”归约（假设结合性），极大提升 GPU/并行性能。你仍可用 fetch_add 获得精确顺序语义。
- 主要应用场景

- 高性能直方图（histogram）构建。
- 并行归约（reduction）、矩阵组装、数据分析。
- GPU 编程（大幅减少原子操作次数）。
- std::execution::par_unseq 算法内部安全使用原子操作。
- 任何“只更新、不需要旧值”的场景。
- 特性测试宏

cpp

```cpp
#ifdef __cpp_lib_atomic_reductions
// C++26 可用
#endif
```

总结C++26 的 Atomic Reduction Operations（store_add、store_min 等）提供了更高效、更安全的原子归约方式。它与 fetch_min/fetch_max、parallel_scheduler、parallel range algorithms 一起，显著提升了 C++ 在高性能并行计算领域的竞争力，尤其适合现代 CPU/GPU 混合编程。与之前 fetch_min/fetch_max 的关系：store_min/store_max 是本提案对 min/max 的补充（不返回旧值的版本）。需要 store_* 与 std::execution、async_scope 或 GPU 场景的完整示例吗？或者与其他原子特性的对比？