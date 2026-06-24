C++26 并行 Ranges 算法（Parallel Range Algorithms）（提案 P3179R9，已被纳入 C++26）。这是 C++ Ranges（C++20）与 C++17 并行算法（Parallel Algorithms）的重要融合，终于让 std::ranges 算法支持执行策略（Execution Policy），从而实现结构化、现代风格的并行处理。1. 为什么需要这个特性？

- C++17 并行算法使用迭代器对（begin/end），不够现代。
- C++20 std::ranges 算法优雅、支持管道（|）、投影（projection）、sentinel 等，但缺少并行支持。
- 开发者希望同时享受 Ranges 的表达力 + 并行性能。

C++26 通过新增以执行策略为首参的重载，完美填补了这一空白。2. 基本用法

cpp

```cpp
#include <ranges>
#include <algorithm>
#include <execution>
#include <vector>
#include <iostream>

int main() {
    std::vector<int> v(1'000'000);
    std::ranges::iota(v, 0);

    // C++26 并行 Ranges 算法（推荐写法）
    std::ranges::for_each(std::execution::par, v, [](int& x) {
        x *= 2;
    });

    // 支持管道风格（部分算法）
    auto sum = std::ranges::reduce(std::execution::par, v, 0LL,
                                   std::plus<>{});

    // 带投影
    struct Person { std::string name; int age; };
    std::vector<Person> people = ...;
    std::ranges::sort(std::execution::par, people, {}, &Person::age);
}
```

3. 接口设计特点

- 执行策略作为第一个参数：std::execution::seq、par、par_unseq 等（以及实现定义的策略）。
- 大多数 std::ranges 算法 都新增了并行重载（非修改 + 修改算法）。
- 约束更严格：
  - 大多要求 random_access_range（因为并行需要高效随机访问）。
  - 输出范围通常也要求 sized_random_access_range。
- 返回值：
  - 大部分保持与串行 Ranges 算法一致（例如返回 borrowed_iterator_t 或 result 对象）。
  - for_each 只返回迭代器（不再返回函数对象，因为并行中难以有效累积状态）。

示例对比：

cpp

```cpp
// C++17（迭代器风格）
std::for_each(std::execution::par, v.begin(), v.end(), f);

// C++20（Ranges，串行）
std::ranges::for_each(v, f);

// C++26（Ranges + 并行，最推荐）
std::ranges::for_each(std::execution::par, v, f);
```

4. 支持的算法（部分列表）

- 非修改：for_each、any_of、all_of、none_of、find、count、reduce、transform_reduce 等。
- 修改：copy、move、fill、transform、sort、stable_sort、partial_sort、nth_element、unique 等。
- 集合算法：merge、set_union 等（部分受限）。

不支持 或有特殊限制的算法（如某些需要严格顺序的）会在提案中明确说明。5. 与 std::execution（Sender/Receiver）的关系

- 并行 Ranges 算法基于 C++17 执行策略（简单、易用）。
- 而 C++26 的 std::execution::parallel_scheduler + bulk 等提供更强大、结构化的并行方式。
- 两者互补：简单场景用 ranges::xxx(par)，复杂异步/结构化并发用 Sender/Receiver。
- 性能与注意事项

- 要求：元素类型满足并行安全（无数据竞争、异常行为符合策略要求）。
- par_unseq 允许向量化，但函数必须是 noexcept 或严格限制异常。
- 实现可能使用线程池（与 get_parallel_scheduler() 可能共享底层资源）。
- 随机访问是关键：链表等非随机访问容器无法高效并行。

总结C++26 的 Parallel Range Algorithms 让 std::ranges 成为真正“生产级”并行工具，极大提升了代码可读性、组合性和现代感。它是 Ranges、并行算法、结构化并发三者融合的重要一步。推荐在新代码中优先使用 std::ranges::xxx(std::execution::par, range, ...) 风格。需要具体算法的详细示例（如 transform、sort、reduce 与管道结合）或其他 C++26 Ranges 特性吗？