**C++26: More constexpr Containers and Adaptors** 是 C++26 中 **constexpr** 支持的重大扩展，通过提案 **P3372R3**（Hana Dusíková）实现。

### 背景

在 C++20 中，只有少数容器支持 `constexpr`：
- `std::vector<T>`
- `std::basic_string` / `std::string`
- `std::array`
- `std::span`、`std::mdspan`

C++23 有所增加，但仍不完整。

**C++26** 将情况彻底反转：**几乎所有标准容器和容器适配器** 都标记为 `constexpr`，极大提升了编译时编程的能力。

### 主要变更（P3372R3）

- **支持 `constexpr`** 的容器和适配器：
  - **顺序容器**：`std::deque`、`std::list`、`std::forward_list`
  - **关联容器**：`std::map`、`std::set`、`std::multimap`、`std::multiset`
  - **无序容器**：`std::unordered_map`、`std::unordered_set`、`std::unordered_multimap`、`std::unordered_multiset`
  - **扁平容器**（C++23 引入）：`std::flat_map`、`std::flat_set`、`std::flat_multimap`、`std::flat_multiset`
  - **容器适配器**：`std::stack`、`std::queue`、`std::priority_queue`

- **例外**：
  - `std::hive`（C++26 新容器）**不**支持 `constexpr`（因为其实现过于复杂）。

- **约束**：
  - 容器元素类型 `T` 必须满足 `constexpr` 构造/析构/复制要求（`is_trivially_copyable` 不是必须的，但必须支持常量求值）。
  - 动态内存分配在常量求值中必须**配对释放**（与 `constexpr` `new`/`delete` 规则一致）。

### Feature Test Macros（特性测试宏）

每个容器都有独立的宏，例如：

```cpp
#if __cpp_lib_constexpr_deque >= 202502L
// std::deque 是 constexpr 的
#endif

// 类似的有：
__cpp_lib_constexpr_list
__cpp_lib_constexpr_map
__cpp_lib_constexpr_set
__cpp_lib_constexpr_unordered_map
__cpp_lib_constexpr_stack
__cpp_lib_constexpr_queue
__cpp_lib_constexpr_priority_queue
// ... 等
```

### 示例（代码测试不通过）

```cpp
#include <vector>
#include <map>
#include <set>
#include <deque>
#include <stack>
#include <string>
#include <iostream>

constexpr auto build_data() {
    std::map<std::string, int> m;
    m["apple"] = 5;
    m["banana"] = 3;

    std::deque<int> d{1, 2, 3, 4};
    d.push_back(42);

    std::stack<int> s;
    s.push(10);
    s.push(20);

    return std::tuple{m, d, s};
}

int main() {
    constexpr auto [m, d, s] = build_data();
    std::cout << m.at("apple") << '\n';  // 5
}
```

在 `constexpr` 函数中，你现在可以自由使用插入、删除、查找、排序等几乎所有操作。

### 实际意义

- **编译时数据准备**：可以在编译期构建复杂的查找表、配置树、静态资源等，运行时零开销。
- **代码复用**：同一份容器代码可在编译时（测试、生成常量数据）和运行时使用。
- **元编程能力** 大幅提升，尤其配合 `constexpr` 算法、`std::string`、`std::vector` 等。
- **测试**：可以在 `static_assert` 中直接构造和验证容器内容。

### 注意事项

- **性能**：常量求值中的容器操作通常比运行时慢很多，仅用于编译期。
- **实现状态**：GCC 16.1+、Clang 19+ 等已开始支持大部分特性。
- **Allocator**：默认 `std::allocator` 在 `constexpr` 中可用；`std::pmr` 版本支持有限。

这是 C++26 “constexpr 大扩张”中最重要的一部分，让编译时编程真正接近运行时编程的表达力。

更多细节可参考：
- 提案 [P3372R3](https://wg21.link/P3372)
- cppreference: C++26 页面（More constexpr containers and adaptors）

需要具体某个容器（如 `constexpr std::unordered_map` 或 `std::priority_queue`）的详细示例吗？