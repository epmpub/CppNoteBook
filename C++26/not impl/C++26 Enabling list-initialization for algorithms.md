#### C++26 Enabling list-initialization for algorithms

备注：2026/06/18 GCC16.1 未实现

（提案 P2248R8 + 后续补丁 P3217R0、P3787R2 等）是一个重要的易用性改进。

它让标准算法（尤其是 std::ranges 版本）可以直接接受 braced-init-list（{1, 2, 3}）作为输入，而不需要先创建一个具名容器。之前的问题（C++23 及更早）

```cpp
std::vector<int> v = {3, 1, 4, 1, 5};

// 必须这样写
std::ranges::sort(v);

// 或者临时创建
std::ranges::sort(std::vector{3, 1, 4, 1, 5});  // 丑陋且有额外开销
```

很多算法无法直接传入 {...}，导致代码啰嗦。C++26 的新能力现在可以直接这样写：

```cpp
#include <algorithm>
#include <ranges>
#include <vector>
#include <iostream>

int main() {
    // 1. ranges 算法直接接受 initializer_list
    std::ranges::sort({3, 1, 4, 1, 5});                    // OK!
    
    auto sorted = std::ranges::sort({10, 7, 8, 9, 1});    // 返回 borrowed subrange

    // 2. 很多接受 range 的算法现在都支持
    std::ranges::for_each({1, 2, 3, 4}, [](int x) {
        std::cout << x << ' ';
    });

    if (std::ranges::contains({1, 2, 3, 4, 5}, 3)) {
        std::cout << "found!\n";
    }

    // 3. 传统算法也得到改进（部分）
    std::sort(std::vector{5, 4, 3, 2, 1}.begin(), std::vector{5, 4, 3, 2, 1}.end()); // 仍需小心
}
```

支持的主要算法（部分列表）

- std::ranges::sort / stable_sort / partial_sort
- std::ranges::find / find_if / contains
- std::ranges::min / max / minmax
- std::ranges::for_each
- std::ranges::count / count_if
- std::ranges::copy、transform 等很多算法
- 以及一些 uninitialized_ 系列算法（后续补丁）

特性测试宏

```cpp
#if __cpp_lib_algorithm_default_value_type >= 202403L  // 具体日期请查最新 cppreference
// 支持 list-initialization for algorithms
#endif
```

为什么要做这个改进？

- 极大提升代码简洁性和可读性。
- 减少不必要的临时变量和内存分配。
- 让 C++ 的算法接口更接近脚本语言（如 Python）的自然表达方式。
- 修复长期以来 std::initializer_list 与算法接口不匹配的问题。

一句话总结：
C++26 让 {1, 2, 3, ...} 可以直接作为参数传给大多数 std::ranges::xxx 算法，写法更加现代、简洁、自然，是 C++26 中最受欢迎的易用性特性之一。需要我给你更多实际使用场景的例子吗？（比如和 views::concat、transform 组合使用）