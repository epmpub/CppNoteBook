**C++26 中 `std::memory_order::consume` 的 Defang 和 Deprecate** 是通过提案 **P3475R2**（*Defang and deprecate `memory_order::consume`*）引入的变化。

### 背景与问题

`memory_order_consume`（C++11 引入）本意是提供一种**比 `acquire` 更弱**的同步语义：

- 只针对**数据依赖**（data dependency / address dependency）进行同步。
- 理论上性能更高（尤其在 ARM、Power 等弱内存模型架构上）。
- 需要配合 `std::kill_dependency()` 和 `[[carries_dependency]]` 属性使用。

**实际问题**（存在十多年）：
- 定义过于复杂，实现难度极高。
- 所有主流编译器（GCC、Clang、MSVC）实际上都**直接把它当成 `memory_order_acquire`** 处理（即提升为更强的同步）。
- 很少有代码能正确、安全地使用它。
- `kill_dependency()` 和 `[[carries_dependency]]` 在实践中几乎无人使用。

### C++26 的变更（Defang + Deprecate）

1. **Deprecate（弃用）**：
   - `std::memory_order::consume` 被标记为 `[[deprecated]]`。
   - 使用它会在编译时产生**弃用警告**（`-Wdeprecated`）。

2. **Defang（弱化 / 去功能化）**：
   - 标准明确规定：**`memory_order_consume` 的行为现在与 `memory_order_acquire` 完全相同**。
   - 释放-消费（release-consume）排序现在**等价于**释放-获取（release-acquire）排序。
   - 相关的“依赖有序”概念被大幅简化或移除。

**cppreference 当前描述**（C++26）：
> `memory_order_consume` (**deprecated in C++26**)  
> Release-consume ordering has the same effect as release-acquire ordering and is deprecated. (since C++26)

### 影响与推荐做法

- **现有代码**：
  - 继续使用 `memory_order_consume` 在功能上是安全的（因为被提升为 acquire），但会收到警告。
  - 建议逐步替换为 `memory_order_acquire`（语义更清晰、更可预测）。

- **新代码**：
  - **直接使用 `memory_order_acquire`**，不要再考虑 `consume`。
  - 在大多数场景下，`acquire` 的性能损失微乎其微，而语义更可靠。

**示例**（推荐写法）：

```cpp
// 旧写法（C++26 起不推荐）
std::atomic<T*> ptr;
T* p = ptr.load(std::memory_order_consume);

// 新推荐写法
T* p = ptr.load(std::memory_order_acquire);
```

### 为什么这么做？

- 简化内存模型，减少实现负担。
- 避免开发者被一个“看起来有用但实际上没用”的特性误导。
- 为未来可能的更强“dependency ordering”机制（如硬件 address dependency）留出空间（但目前不现实）。

**Feature Test**：没有专门的宏，但可以通过 `__cpp_lib_atomic` 或直接检查编译器版本判断。

更多细节可参考：
- 提案 [P3475R2](https://wg21.link/P3475)
- cppreference: [`std::memory_order`](https://en.cppreference.com/w/cpp/atomic/memory_order)

这是 C++26 对原子内存模型的一次“清理”操作，让标准更加务实。如果你有使用 `consume` 的具体代码场景，我可以帮你分析如何迁移。