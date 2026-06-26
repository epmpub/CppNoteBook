**C++26 Freestanding Library Support for `<algorithm>`, `<execution>`, `<memory>`, `<numeric>` 和 `<random>`**

这是 C++26 中**Freestanding（无主机环境）** 支持的重大扩展，主要通过提案 **P2976R1**（*Freestanding Library: `algorithm`, `numeric`, and `random`*）实现。

### 背景

**Freestanding** 实现（嵌入式、内核、实时系统、某些 GPU 环境等）**不允许**依赖操作系统功能（如动态内存分配、文件 I/O、线程等）。之前很多标准库头文件在 freestanding 下不可用或支持极少。

C++26 大幅提升了 freestanding 的可用性，让更多算法和工具能在**无 OS** 环境下使用。

### 具体支持情况（C++26）

| 头文件            | 支持程度                       | 主要新增内容                                                 |
| ----------------- | ------------------------------ | ------------------------------------------------------------ |
| **`<algorithm>`** | **大部分支持**（Freestanding） | 绝大多数非并行算法（如 `sort`、`find`、`copy`、`transform`、`min`、`max`、`for_each` 等） |
| **`<numeric>`**   | **大部分支持**（Freestanding） | `iota`、`accumulate`、`reduce`、`inner_product`、`partial_sum`、`exclusive_scan` 等 |
| **`<memory>`**    | **显著增强**（Freestanding）   | `uninitialized_copy`、`uninitialized_fill`、`destroy`、`construct_at` 等未初始化内存算法；`allocator` 相关部分 |
| **`<random>`**    | **大部分支持**（Freestanding） | 随机数引擎（`mt19937` 等）、分布（`uniform_int_distribution` 等）、`random_device`（实现定义） |
| **`<execution>`** | **部分支持**                   | 执行策略（如 `std::execution::seq`、`par`、`par_unseq`）**但并行重载在 freestanding 下通常被删除（freestanding-deleted）** |

**重要说明**：
- **执行策略（Execution Policies）**：`<execution>` 头文件可用，但**并行版本**（`par`、`par_unseq`）在 freestanding 实现中通常**不要求提供**（可能被删除或空实现）。
- 所有新增支持都带有**特性测试宏**（Feature Test Macros），便于条件编译。

### 特性测试宏（Feature Test Macros）

```cpp
#if __cpp_lib_freestanding_algorithm >= 2025XXL
// <algorithm> 支持
#endif

#if __cpp_lib_freestanding_numeric >= 2025XXL
// <numeric> 支持
#endif

#if __cpp_lib_freestanding_memory >= 2025XXL
// <memory> 支持
#endif

#if __cpp_lib_freestanding_random >= 2025XXL
// <random> 支持
#endif

#if __cpp_lib_freestanding_execution >= 2025XXL
// <execution> 支持
#endif
```

### 示例（在 Freestanding 环境中）

```cpp
#include <algorithm>
#include <numeric>
#include <memory>
#include <random>
#include <array>

constexpr int compute() {
    std::array<int, 10> arr{};
    std::iota(arr.begin(), arr.end(), 1);           // <numeric>

    std::sort(arr.begin(), arr.end());              // <algorithm>

    int sum = std::accumulate(arr.begin(), arr.end(), 0);

    // 未初始化内存
    alignas(alignof(int)) unsigned char buffer[sizeof(int)*5];
    auto* p = std::start_lifetime_as<int>(buffer);
    std::uninitialized_fill(p, p+5, 42);            // <memory>

    return sum;
}

int main() {
    std::mt19937 gen{42};                           // <random>
    std::uniform_int_distribution<> dist(1, 100);

    // ... 使用算法和随机数 ...
}
```

### 实际意义

- **嵌入式 / 内核开发者**：现在可以更方便地在 freestanding 环境中使用标准算法、随机数和内存工具，而无需自己重新实现或依赖第三方库。
- **代码可移植性**：同一份算法代码可在 hosted 和 freestanding 环境下复用（通过 `#ifdef` 或特性测试宏）。
- **与 constexpr 结合**：这些头文件中的很多函数同时支持 `constexpr`，进一步增强编译时能力。

这是 C++26 “More Freestanding Facilities” 系列提案的一部分（继 `<charconv>`、`<functional>`、`<iterator>` 等之后）。

**参考**：
- 提案：**P2976R1**
- cppreference：C++26 Freestanding 页面

需要某个具体头文件（如 `<random>` 中的引擎/分布）的详细列表，或 freestanding 下的限制示例吗？随时告诉我！