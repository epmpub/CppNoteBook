**C++26 Library support for observable checkpoints**

 是通过 **P1494R5**（*Partial program correctness*）引入的核心语言/库特性，并由 **P3641R0** 完善（重命名为 `std::observable_checkpoint` 并添加特性测试宏）。

### 背景

C++ 的 **as-if 规则**（as-if rule）允许编译器在不改变**可观察行为**（observable behavior）的前提下进行任意优化。但**未定义行为（UB）** 会导致“时间旅行”优化（time-travel optimizations）：编译器可能假设 UB 永远不会发生，从而删除或重排前面的代码。

**Observable checkpoints**（可观察检查点）解决了这个问题：它在程序中建立一个“栅栏”，限制 UB 的影响范围。

### `std::observable_checkpoint()`

```cpp
#include <utility>   // C++26

[[noreturn]] void std::unreachable() noexcept;
void std::observable_checkpoint() noexcept;   // C++26
```

- **作用**：在调用点**建立一个 observable checkpoint**。
- **语义**：告诉编译器——在此点之前的所有代码，其可观察行为必须被保留（不能被 UB 后面的假设“时间旅行”优化掉）。
- **效果**：`noexcept`、`无副作用`（除了建立检查点），编译器通常生成极少的代码（可能只是一个空调用或内存屏障）。

### 主要用途

1. **限制 UB 的传播**：
   ```cpp
   int foo(int x) {
       if (x < 0) {
           std::observable_checkpoint();  // 在 UB 之前建立检查点
           // ... 一些可能导致 UB 的代码
       }
       return 42 / x;   // 潜在 UB（除以0）
   }
   ```

2. **与 Contracts（C++26）配合**：
   - Contract assertions（`pre`、`post`、`assert`）在 `observe` 语义下会自动成为 observable checkpoints。
   - 这保证了 contract violation handler 的行为不会被后续 UB 影响。

3. **调试 / 安全性**：
   - 在关键检查点放置，防止编译器因假设后续 UB 而优化掉前面的重要代码（如安全检查、日志等）。
   - 类似于 `std::unreachable()`，但方向相反（保护前面的代码）。

### 与 `as-if` 规则的关系（C++26 更新）

C++26 正式引入了 **defined prefix** 的概念：

- 程序可以包含 observable checkpoints。
- 对于每个 UB 操作 `U`，如果存在检查点 `CP` 使得 `CP` 在 `U` 之前发生，则 `U` 之前的操作属于 **defined prefix**。
- 编译器只能在 defined prefix 内进行优化。

### Feature Test Macro

```cpp
#if __cpp_lib_observable_checkpoint >= 2025XXL
// 支持 std::observable_checkpoint
#endif
```

### 总结

- `std::observable_checkpoint()` 是 C++26 **安全性与正确性**改进的重要组成部分。
- 它与 **Contracts**、**Library Hardening** 一起，大幅提升了标准库和用户代码在面对 UB 时的可预测性。
- 特别适合系统编程、安全关键代码、调试场景，以及需要严格控制优化行为的开发者。

**参考**：
- 提案：P1494R5 + P3641R0
- cppreference：C++26 页面（Observable checkpoints）

需要具体使用示例（与 Contracts 结合、调试 UB 等）还是与其他安全特性的对比？随时告诉我！