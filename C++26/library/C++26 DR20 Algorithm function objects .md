

DR20: Algorithm function objects

测试代码：

```c
#include <ranges>      // views, range concepts
#include <algorithm>   // 必须包含！ranges 算法如 sort, distance 等
#include <vector>
#include <iostream>
#include <functional>

int main()
{
    // 1. range of ranges 示例
    std::vector<std::vector<int>> ranges = {
        {1, 2, 3},
        {4, 5},
        {6, 7, 8, 9}
    };

    // C++26 中合法
    auto v = ranges | std::views::transform(std::ranges::size);
    std::cout << "Sizes (using size): ";
    for (auto s : v) std::cout << s << ' ';
    std::cout << '\n';

    auto w = ranges | std::views::transform(std::ranges::distance);
    std::cout << "Sizes (using distance): ";
    for (auto s : w) std::cout << s << ' ';
    std::cout << '\n';

    // 2. Algorithm function object 存储示例
    std::vector<std::function<void(std::vector<int>&)>> funcs;

    funcs.push_back(std::ref(std::ranges::sort));   // 现在可以正常使用

    std::vector<int> numbers = {5, 3, 8, 1, 9};
    funcs[0](numbers);   // 调用 std::ranges::sort

    std::cout << "After sort: ";
    for (int n : numbers) std::cout << n << ' ';
    std::cout << '\n';

    return 0;
}
```



（提案 P3136R1）是 C++26 中的一个缺陷报告修复（Defect Report against C++20），它对 std::ranges 中的算法进行了规范化（re-specification）。背景：什么是 “Niebloid”？在 C++20 引入 Ranges 时，std::ranges::find、std::ranges::sort、std::ranges::distance 等算法被非正式地称为 “niebloids”（niebloid = “non-inlinable entity” 的戏称，由 Eric Niebler 提出）。它们的特殊行为包括：

- 抑制 ADL（Argument-Dependent Lookup）：防止找到 std 命名空间中的经典算法（如 std::find）。
- 不能显式指定模板参数（ranges::find<int>(...) 是未指定的）。
- 实现上通常是函数对象（function object），而不是真正的函数模板。

但标准原文却把它们描述成“函数模板”，这与实际实现不符，导致了一些规范上的不精确和限制。C++26 的修改（P3136R1）正式引入 “Algorithm function objects”（算法函数对象）这一概念，并将 std::ranges 中的绝大部分算法明确指定为这种对象，而不再假装它们是函数模板。核心变化

1. 新增规范术语（在 [conventions] 章节）：
   - 定义 algorithm function object 为一种特殊的 customization point object（CPO）。
   - 它由一组重载的函数模板实现，但名称本身指代的是函数对象（一个可调用对象）。
2. 明确了以下行为（现在是正式保证）：
   - 支持拷贝（semiregular）。
   - 可以作为值传递给 std::views::transform、std::ranges::for_each 等。
   - 可以用 std::ref() 包装。
   - 行为更接近真正的函数对象，而不是“伪函数模板”。

实际影响与好处

cpp

```cpp
// C++23 可能有问题或需要变通
auto v = std::views::transform(ranges, std::ranges::size);           // OK（size 是 CPO）
auto w = std::views::transform(ranges, std::ranges::distance);       // 以前可能不行

// C++26：明确合法且推荐
auto x = std::views::transform(data, std::ranges::distance);

// 也可以这样用
std::vector<std::function<...>> funcs;
funcs.push_back(std::ref(std::ranges::sort));   // 现在更有保障
```

主要改进：

- 允许直接把 std::ranges::xxx 作为函数对象传递，而无需写 lambda 包装。
- 消除实现与标准描述之间的不一致。
- 提升 Ranges 算法作为“一等公民”函数对象的可用性，尤其在函数式编程风格的代码中。

总结

- 这是一个清理性质的 DR，不是新增特性，而是把 C++20 引入的 Ranges 算法的实现模型正式化。
- “Niebloid” 这个非正式术语逐渐被 “Algorithm function object” 取代。
- 提案作者：Tim Song（标准库专家）。

一句话概括：C++26 承认并正式规范了 std::ranges 算法其实一直是函数对象，让代码更干净、可组合性更强，也为未来扩展（如并行 ranges 算法）铺平了道路。 更多细节可参考：

- 提案：[P3136R1 - Retiring niebloids](https://wg21.link/P3136R1)
- cppreference C++26 特性表中标记为 DR20: Algorithm function objects。

