std::is_sufficiently_aligned（C++26） 是一个简单的工具函数，用于在运行时安全地检查指针的对齐（alignment）是否满足指定要求，主要服务于 std::aligned_accessor 和 std::assume_aligned 等过对齐场景。 

en.cppreference.com

声明

cpp

```cpp
#include <memory>

template <std::size_t N, class T>
bool is_sufficiently_aligned(T* ptr);   // (since C++26)
```

- N：期望的最小字节对齐值，必须是 2 的幂。
- T：指针指向的对象类型（忽略 cv 限定）。

功能说明

- 作用：检查 ptr 指向的对象是否至少按 N 字节对齐。
- 返回值：如果对齐满足，返回 true；否则返回 false。
- 异常：不抛出任何异常（noexcept 语义）。
- 未定义行为：如果 ptr 不指向一个有效的 T 类型对象（忽略 cv），则行为未定义。

可能实现（Cppreference 示例）

cpp

```cpp
template <std::size_t N, class T>
bool is_sufficiently_aligned(T* ptr) {
    return std::bit_cast<std::uintptr_t>(ptr) % N == 0;
}
```

它本质上是将指针地址转换为整数后检查是否能被 N 整除。主要用途与 std::aligned_accessor<ElementType, ByteAlignment> 配合使用：

cpp

```cpp
void compute(mdspan<float, dims<1>> x) {
    constexpr std::size_t align = 4 * alignof(float);  // 例如 64 字节对齐

    if (std::is_sufficiently_aligned<align>(x.data_handle())) {
        auto acc = std::aligned_accessor<float, align>{};
        auto aligned_mdspan = mdspan{x.data_handle(), x.mapping(), acc};
        // 使用 aligned_mdspan 享受更好的向量化优化
    } else {
        // 回退到普通 default_accessor
    }
}
```

它解决了手动检查对齐时的痛点：用户不需要自己处理指针到整数的转换、符号、位宽等问题，标准库提供了可移植、安全的检查方式。与相关特性的关系

- std::assume_aligned<N>(ptr)：告诉编译器指针已对齐（有 precondition）。is_sufficiently_aligned 可作为其前置检查，避免 UB。
- std::aligned_accessor：mdspan 的访问器策略，使用 assume_aligned 进行内部访问。is_sufficiently_aligned 正是为其设计的辅助工具。
- 特性测试宏：__cpp_lib_is_sufficiently_aligned (202411L)。

注意事项

- 这是一个运行时检查，常用于动态决定是否使用过对齐路径。
- 对齐检查是precondition，违反 assume_aligned 或构造 aligned_accessor 仍可能导致 UB。
- 它与 alignof（编译期类型对齐）互补：一个查类型要求，一个查具体指针地址。

参考：

- cppreference: [`std::is_sufficiently_aligned`](https://en.cppreference.com/cpp/memory/is_sufficiently_aligned)
- 提案：[P2897R7](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2024/p2897r7.html)（aligned_accessor 提案，同时引入此函数）

这是 C++26 为高性能代码（尤其是 SIMD、线性代数）提供的又一个小而实用的安全工具。