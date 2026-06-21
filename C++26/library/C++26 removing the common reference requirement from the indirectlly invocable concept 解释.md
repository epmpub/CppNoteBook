C++26 removing the common reference requirement from the indirectlly invocable concept 解释

std::indirectly_invocable 概念中移除 common_reference 要求 是 C++26 中的一个重要修复（Defect Report against C++20），提案为 P2997R1（Barry Revzin, Tim Song）。问题背景（为什么需要移除？）在 C++20/23 中，Ranges 库中的几个间接调用概念（indirectly invocable concepts）包含了多余且有害的要求：

- indirectly_unary_invocable
- indirectly_regular_unary_invocable
- indirect_unary_predicate
- indirect_binary_predicate
- indirect_equivalence_relation
- indirect_strict_weak_order
- 等

这些概念除了要求可调用性外，还强制要求：

cpp

```cpp
// 之前的要求（已移除）
invoke_result_t<F&, iter_reference_t<I>> 
    && common_reference_with<...>  // 或 iter_common_reference_t 相关检查
```

核心问题：要求 invoke_result_t 与 iter_common_reference_t<I>（即迭代器 reference 类型和 value_type& 的公共引用类型）之间存在 common_reference。这个要求经常错误地拒绝合法代码，尤其是在：

- 代理引用（proxy reference）迭代器（如 std::vector<bool>::reference）
- 返回值类型与迭代器 reference 类型不完全匹配但实际可用的自定义迭代器
- 某些移动语义或转换场景

它没有提供有意义的安全或正确性保证，反而成了不必要的障碍。C++26 的修改直接移除了所有 indirectly_* 概念中与 common_reference / iter_common_reference_t 相关的调用要求。修改后，这些概念现在只关注真正的可调用性（invocability）和谓词/关系的基本要求，而不再强加引用类型之间的 common_reference 约束。实际影响示例（来自提案）

cpp

```cpp
// C++23：可能不通过（取决于迭代器类型）
// C++26：一定通过
auto r = std::ranges::find_if(v, [](auto&& x) { return some_condition(x); });
```

以前某些合法的 lambda 或可调用对象会被概念检查拒绝，现在可以正常使用。主要受影响的概念

| 概念                               | C++23 行为               | C++26 行为 |
| ---------------------------------- | ------------------------ | ---------- |
| indirectly_unary_invocable         | 有 common_reference 要求 | 移除该要求 |
| indirectly_regular_unary_invocable | 有 common_reference 要求 | 移除该要求 |
| indirect_unary_predicate           | 有 common_reference 要求 | 移除该要求 |
| indirect_binary_predicate          | 有 common_reference 要求 | 移除该要求 |
| indirect_strict_weak_order 等      | 有 common_reference 要求 | 移除该要求 |

总结

- 这是一个清理性质的 DR（Defect Report），作为 C++20 的缺陷被回溯修复。
- 不改变正确代码的行为，只让更多合理代码能通过概念检查。
- 极大改善了 Ranges 算法与自定义迭代器、代理引用、复杂 lambda 的兼容性。
- 主要库实现（libstdc++、libc++、MSVC STL）已作为 C++20 的 DR 实现。

一句话概括：C++26 移除了 Ranges 间接调用概念中一个“过于严苛且无实际价值”的类型约束，让 std::ranges 算法更加易用和灵活。更多细节可参考提案：P2997R1。需要我给出前后概念定义的代码对比吗？