C++26 std::views::indices 解释（提案 P3060R3，已被纳入 C++26）。这是 Ranges 库中一个小而实用的新视图适配器，主要目的是提供更方便、语义清晰的方式生成从 0 开始的整数序列（常用于索引）。1. 动机在 C++20/23 中，生成索引最常见的方式是 std::views::iota(0, n)。虽然功能足够，但有以下问题：

- 必须显式写起始值 0，很多情况下这是多余的。
- 语义不明显：iota(0, n) 到底是生成索引还是普通数值序列？
- 代码可读性稍差，尤其在复杂管道中。

std::views::indices(n) 直接解决这些问题，提供零起点、半开区间 [0, n) 的整数视图。2. 接口

cpp

```cpp
namespace std::ranges::views {
    inline constexpr /* unspecified */ indices = /* unspecified */;
}
```

用法：

cpp

```cpp
constexpr auto indices(T bound);   // T 是 integer-like 类型
```

- 返回一个 iota_view（底层仍是 std::ranges::iota_view<...>）。
- 生成序列：0, 1, 2, ..., bound-1。
- 如果 bound <= 0，得到空范围。
- 支持管道风格（pipeable）。
- 使用示例

cpp

```cpp
#include <ranges>
#include <vector>
#include <iostream>

int main() {
    std::vector<std::string> fruits = {"apple", "banana", "cherry"};

    // C++26 新方式（推荐）
    for (auto i : std::views::indices(fruits.size())) {
        std::cout << i << ": " << fruits[i] << '\n';
    }

    // 与其他 views 组合（非常优雅）
    auto indexed = std::views::indices(10)
                 | std::views::transform([](int i) { return i * i; });

    for (auto sq : indexed) {
        std::cout << sq << ' ';  // 0 1 4 9 16 25 36 49 64 81
    }
}
```

与旧方式对比：

cpp

```cpp
// C++23 及之前
for (auto i : std::views::iota(0u, fruits.size())) { ... }   // 必须写 0

// C++26
for (auto i : std::views::indices(fruits.size())) { ... }    // 更清晰
```

4. 与相关特性的关系

| 特性                             | 用途                       | 区别                        |
| -------------------------------- | -------------------------- | --------------------------- |
| std::views::indices(n)           | 生成 [0, n) 索引           | 专为索引设计，默认从 0 开始 |
| std::views::iota(start, end)     | 生成任意 [start, end) 序列 | 更通用，可指定起点          |
| std::views::enumerate(r) (C++23) | 同时得到索引 + 元素        | 返回 tuple<index, value>    |

推荐场景：

- 遍历容器时需要索引（代替传统 for (size_t i = 0; ...)）。
- 生成网格坐标（views::indices(rows) | views::transform(...)）。
- 与 zip、transform 等组合实现并行/向量化友好代码。
- 需要 constexpr 友好整数序列的元编程场景。
- 特性测试宏

cpp

```cpp
#ifdef __cpp_lib_ranges_indices
// C++26 特性可用
#endif
```

值通常为 202506L（具体以最终标准为准）。总结std::views::indices 是一个语法糖 + 语义改进，让“生成索引序列”这个极其常见的操作变得更简洁、直观。它延续了 C++ Ranges “视图优先、组合优先”的设计哲学，是 C++26 Ranges 改进中的一员（与 concat、cache_latest 等并列）。虽然功能可以用 iota(0, n) 完全替代，但可读性显著提升，推荐在新代码中优先使用 views::indices。需要 views::indices 与 enumerate、zip、chunk 等组合的完整示例吗？