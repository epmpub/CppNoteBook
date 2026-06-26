std::optional 的 Ranges 支持 是 C++26 的重要改进（提案 P3168），让 std::optional<T> 正式成为 Ranges 库 中的一个 View（大小为 0 或 1 的 range）。核心变化从 C++26 开始：

- 
  std::optional<T>符合 ranges::range 的概念。
- 
  std::optional<T>符合 ranges::view 的概念（通过 ranges::enable_view<std::optional<T>> = true 来实现）。
- 你可以直接对 std::optional 使用范围 for 循环、std::ranges 算法、views pipeline（| 操作符）等。

这体现了函数式编程中的常见理念：optional 本质上就是一个最多包含 0 或 1 个元素的序列。新增成员（迭代器支持）

cpp

```cpp
// C++26 新增
constexpr iterator begin() noexcept;
constexpr const_iterator begin() const noexcept;

constexpr iterator end() noexcept;
constexpr const_iterator end() const noexcept;
```

- 如果 optional 有值（has_value() == true），则 begin() 指向该值，end() 是 past-the-end。
- 如果为空，则 begin() == end()（空范围）。

使用示例

cpp

```cpp
#include <optional>
#include <ranges>
#include <vector>
#include <iostream>
#include <algorithm>

int main() {
    std::optional<int> opt = 42;
    std::optional<int> empty;

    // 1. 范围 for 循环
    for (int x : opt) {
        std::cout << x << '\n';     // 输出 42
    }
    for (int x : empty) {
        // 不会执行
    }

    // 2. 与 ranges 算法配合
    auto vals = std::vector<std::optional<int>>{{1}, {}, {3}, {4}, {}};

    auto sum = std::ranges::fold_left(vals | std::views::join, 0, std::plus{});
    // sum = 1 + 3 + 4 = 8
    std::cout << sum << "\n";

    // 3. views pipeline（非常优雅）
    auto result = vals
        | std::views::filter([](auto& o){ return o.has_value(); })
        | std::views::transform([](auto& o){ return *o * 10; })
        | std::ranges::to<std::vector>();
    for(auto &i : result)
        std::cout << i << " ";

    // result = {10, 30, 40}
}

```

与之前方式的对比

| 方式               | C++23 及以前                        | C++26                                   |
| ------------------ | ----------------------------------- | --------------------------------------- |
| 检查是否有值       | 如果 (opt) 或如果 (opt.has_value()) | 可直接 for (auto v : opt)               |
| 提取值后处理       | 手动 if + *opt                      | 可直接用于 ranges pipeline              |
| 多个 optional 处理 | 嵌套 if 或 手动循环                 | views::join、transform 等极为简洁的用法 |
| 代码简洁度         | 一般                                | 显著提升                                |

设计决策

- 为什么直接让 optional 成为 view，而不是新增 views::maybe？
  委员会最终认为：引入新类型会增加复杂性，而 optional 本身已经非常接近“0或1个元素的容器”，直接增强它更简洁、一致。
- std::optional<T> 仍是 View（轻量、不拥有额外状态），拷贝/移动语义不变。
- 禁用了 std::format 的范围格式化支持（range_format::disabled），避免歧义。

适用场景

- 函数式/声明式风格 的代码（ranges pipeline）。
- 处理一系列可能为空值的计算（例如查找结果、数据库查询、配置项等）。
- 与 std::ranges::join、transform、filter 等组合使用时特别强大。
- 替代以前常见的 if (opt) { use(*opt); } 模式。

总结：C++26 把 std::optional 从“可能有值的包装器”提升为 Ranges 生态的一等公民，极大提升了它在现代 C++（尤其是 ranges-heavy 代码）中的表达力和易用性。更多细节可参考：

- 
  cppreference：std::optional
- 
  提案：P3168R2：为 std::optional 添加对 Range 类型的支持

需要我给你更多实际的 ranges pipeline 示例吗？