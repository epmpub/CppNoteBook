### C++26 DR23: std::basic_const_iterator should follow its underlying type's convertibility

C++26 DR23（对应提案 P2836R1） 的核心修复是：让 std::basic_const_iterator<I> 遵循其底层迭代器 I 的可转换性（convertibility），主要添加了从 basic_const_iterator 到“常量迭代器”的隐式转换操作符。 



背景与问题C++23 引入了 std::basic_const_iterator（以及 std::const_iterator、ranges::cbegin 等），目的是让 ranges::cbegin / ranges::cend 始终返回真正的常量迭代器（即使底层是可变视图/范围）。问题示例（C++23 原行为）：

cpp

```cpp
auto f = [](std::vector<int>::const_iterator i) {};  // 接受 vector 的 const_iterator

auto v = std::vector<int>{};

// 普通容器：正常
auto i1 = std::ranges::cbegin(v);  // vector<int>::const_iterator
f(i1);  // OK，因为 vector::iterator 可隐式转为 vector::const_iterator

// 某些 range（如 take_while）：
auto t = v | std::views::take_while([](int x){ return x < 100; });
auto i2 = std::ranges::cbegin(t);  // 返回 basic_const_iterator<vector<int>::iterator>
f(i2);  // ❌ 编译错误！
```

虽然 basic_const_iterator<vector::iterator> 在语义上“等价”于常量迭代器，但它没有转换到 vector::const_iterator 的能力，导致代码在 C++23 中从 C++20 回归后意外无法编译。 

open-std.org

这是 P2278（让 cbegin 始终返回 const iterator）引入的回归问题。解决方案（DR23 的修复）在 std::basic_const_iterator<I> 中添加转换操作符：

cpp

```cpp
template<not-a-const-iterator CI>  // CI 不是 basic_const_iterator 的特化
    requires /*constant-iterator*/<CI> && std::convertible_to<const I&, CI>
constexpr operator CI() const&;

template<not-a-const-iterator CI>
    requires /*constant-iterator*/<CI> && std::convertible_to<I, CI>
constexpr operator CI() &&;
```

- const& 版本：转发 current_（底层迭代器）的 const 左值。
- && 版本：转发右值（move）。
- 只转换到常量迭代器（constant-iterator 概念），避免意外转为可变迭代器。
- 使用 not-a-const-iterator 防止递归或歧义。

效果：现在 basic_const_iterator<I> 可以隐式转换为任何 I 能转换到的常量迭代器类型，从而恢复了预期的兼容性。 

cppreference.com

为什么是 Defect Report（DR）？

- 作为针对 C++23 的 DR 处理（许多实现已作为 C++23 的 bug 修复回溯）。
- C++26 正式纳入，但鼓励库实现者尽快 backport 到 C++23。 libcxx.llvm.org

实际影响

- 正面：提高了 ranges::cbegin / std::const_iterator 的可用性，与传统容器 const_iterator 的行为更一致。ranges 代码在混合使用容器迭代器和 view 迭代器时更少 surprise。
- 实现：已有多家 STL（如 libc++、Microsoft STL）实现此修复。
- cppreference 已更新，basic_const_iterator 文档中增加了 operator constant-iterator。 cppreference.com

一句话总结：这个 DR 修复了“basic_const_iterator 虽然语义上是 const 的，但缺少必要的类型转换，导致与现有 const_iterator 不兼容”的问题，让 const 迭代器在 ranges 世界里真正“ behave like const”。这是 C++23 ranges 改进后的一个重要兼容性补丁。 如果你有具体代码示例或想看完整 wording，可以再提供，我可以进一步解释。