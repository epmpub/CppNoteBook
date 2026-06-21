## C++26 Deprecating std::is_trivial and std::is_trivial_v 

std::is_trivial 和 std::is_trivial_v 在 C++26 中被弃用（Deprecated），提案为 P3247R2（Deprecate the notion of trivial types）。为什么弃用？std::is_trivial<T> 检查的是**“trivial type”**概念，即：

- 类型是 trivially copyable（可平凡拷贝），并且
- 类型是 trivially default constructible（可平凡默认构造）。

这个组合在实践中很少被同时需要，反而容易导致误用。提案认为：

- “trivial type” 这个笼统的概念已经过时（类似 C++20 弃用的 std::is_pod）。
- 大多数实际场景只需要检查其中一项，而不是两项一起。
- 继续保留这个 trait 会误导开发者使用不精确的检查。

因此，标准决定弃用 std::is_trivial，鼓励开发者使用更精确、更细粒度的 type traits。推荐的替代方案根据具体需求，选择以下 trait：

| 需求场景                          | 推荐使用的 trait                                            | 说明     |
| --------------------------------- | ----------------------------------------------------------- | -------- |
| 需要 memcpy / memmove 安全        | std::is_trivially_copyable<T>                               | 最常用   |
| 需要默认构造不做事情              | std::is_trivially_default_constructible<T>                  | -        |
| 需要拷贝构造是 trivial 的         | std::is_trivially_copy_constructible<T>                     | -        |
| 需要移动构造是 trivial 的         | std::is_trivially_move_constructible<T>                     | -        |
| 需要析构是 trivial 的             | std::is_trivially_destructible<T>                           | -        |
| 同时需要拷贝 + 默认构造（旧语义） | is_trivially_copyable && is_trivially_default_constructible | 手动组合 |

示例：

cpp

```cpp
// C++26 前（已弃用）
if constexpr (std::is_trivial_v<MyType>) { ... }

// C++26 推荐写法
if constexpr (std::is_trivially_copyable_v<MyType> &&
              std::is_trivially_default_constructible_v<MyType>) {
    // 等价于旧的 is_trivial
}
```

实际影响

- 编译期警告：在 C++26 模式下使用 std::is_trivial 或 std::is_trivial_v 会产生 deprecation warning（GCC、Clang、MSVC 已实现）。
- 不影响 ABI：只是编译警告，代码仍能编译通过。
- 未来：很可能在 C++29 或后续版本中移除（remove）。
- 这与 C++20 弃用 std::is_pod 是同一思路：逐步淘汰粗粒度的“老概念”，转向细粒度的现代 traits。

总结C++26 弃用 std::is_trivial 是概念精炼的一部分。
不要再用 std::is_trivial，而是根据具体需求选用 is_trivially_xxx 系列 trait。更多细节可参考：

- 提案：[P3247R2](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2024/p3247r2.html)
- cppreference: [`std::is_trivial`](https://en.cppreference.com/w/cpp/types/is_trivial)（已标记 deprecated in C++26）

需要我给你一个常见使用场景的替换示例吗？