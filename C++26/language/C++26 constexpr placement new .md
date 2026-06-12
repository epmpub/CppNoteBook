#### constexpr placement new

在 C++26 中，通过核心提案 **[P2747R2](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2024/p2747r2.html)**（由 Barry Revzin 提出），C++ 标准实现了 `constexpr` 编译期求值领域的又一项重大突破：**正式支持在常量表达式中使用定位 placement new（即 `new (ptr) T(...)`）**。 [1] 

这一改动彻底解放了编译期容器和底层泛型库的编写方式，解决了 C++20 遗留的编译期初始化痛点。 [1, 2] 

------

## 一、 历史痛点与 C++20 的权宜之计

在 C++20 中，标准虽然允许了编译期动态内存分配（允许在 `constexpr` 中使用 `std::vector` 和 `std::string`），但依然**严格禁止**直接使用 placement new。 [3, 4] 

为了让标准库组件能够在编译期构造对象，C++20 迫不得已引入了一个替代工具 —— **`std::construct_at`**。 [1, 4] 

```cpp
// C++20/C++23 中的标准库做法：
std::construct_at(ptr, arg1, arg2); 
```

## `std::construct_at` 的局限性（为什么要淘汰它？）

`std::construct_at` 只是一个语法上的传话筒，它的功能**极其有限**，完全无法匹配原生的 placement new： [1, 2] 

1. **无法进行默认初始化（Default Initialization）**：它强迫执行值初始化（Value Initialization）。如果你想在编译期分配一段未初始化的缓冲区，然后让某些元素保持默认未初始化状态（以追求性能），`std::construct_at` 做不到。
2. **无法进行列表初始化（List Initialization）**：它不能很好地支持类似 `new (ptr) T{1, 2, 3}` 这样的大括号初始化语法。
3. **无法初始化数组（Array Placement New）**：你不能用它在一块内存上一次性构造一个数组（例如 `new (ptr) int[3]{1, 2, 3}`）。 [1, 5] 

由于这些限制，许多原本可以在编译期运行的现代标准库组件（如 `std::format`、`std::any`、`std::inplace_vector` 甚至某些特定的元编程类）在遇到复杂初始化时被迫放弃 `constexpr`。 [1] 

------

## 二、 C++26 的解决方案：原生的编译期 Placement New

C++26 移除了“Placement new 不能作为常量表达式”的旧限制。在满足生命周期和类型安全的前提下，原生的 `new (ptr) T` 现在可以在 `constexpr` 函数中直接使用。 [4] 

## 核心代码示例

## 1. 支持默认初始化与聚合初始化

```cpp
#include <memory>

struct Agg {
    int x;
    int y;
};

constexpr bool test_placement_new() {
    // 1. 在编译期分配一段未初始化的内存
    std::allocator<Agg> alloc;
    Agg* ptr = alloc.allocate(1);

    // 2. C++26 正式允许：使用大括号列表进行编译期 placement new
    ::new (static_cast<void*>(ptr)) Agg{42, 100};

    bool success = (ptr->x == 42 && ptr->y == 100);

    // 3. 销毁并释放
    std::destroy_at(ptr);
    alloc.deallocate(ptr, 1);

    return success;
}

// 编译期断言通过！
static_assert(test_placement_new());
```

## 2. 支持数组的编译期初始化

这在以前利用 `std::construct_at` 纯属天方夜谭，而 C++26 完美原生支持： [2, 5] 

```cpp
constexpr bool test_array() {
    std::allocator<int> alloc;
    int* p = alloc.allocate(3);

    // C++26：直接在编译期对分配的内存进行数组 placement new
    ::new (p) int[]{1, 2, 3}; 

    bool ok = (p[0] == 1 && p[1] == 2 && p[2] == 3);
    
    alloc.deallocate(p, 3);
    return ok;
}
static_assert(test_array());
```

------

## 三、 为什么直到 C++26 才能实现？

以前不开启此功能，是因为编译器在常量求值（Constant Evaluation）期间，很难追踪一块“被强转为无类型（`void*`）的原始内存”中究竟塞进去了什么类型的对象。 [1, 4] 

C++26 能够成功落地该特性，得益于它拥有两个坚实的底层前置提案：

1. **[P2738R1](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2023/p2308r1.html) (C++26 允许 `constexpr` 期间从 `void\*` 进行显式类型转换)**：解决了内存指针在 `void*` 和 `T*` 之间来回转换的编译期合法性。
2. **严格的常量期类型检查**：虽然 placement new 在运行时极度自由（甚至允许你把一个 `std::string` 强行 new 到一个 `double*` 数组里），但在 `constexpr` 编译期，优化器依然会强制执行安全边界检查。如果你试图在编译期通过 placement new 恶意破坏对象类型或越界，编译器会直接抛出编译错误。 [1, 5, 6] 

------

## 四、 总结与意义

- **大幅简化基础库编写**：诸如 **`std::inplace_vector`（C++26 栈分配可变长数组）**、`std::expected` 等底层高性能容器，终于可以彻底甩掉 `std::construct_at` 的历史包袱，直接用最自然的 C++ placement new 语法实现编译期的高效初始化。
- **统一运行时与编译期行为**：编写泛型代码时，开发者再也不需要写 `if consteval { std::construct_at(...); } else { ::new(...) ; }` 这种割裂的分支代码，一套原生语法即可通吃运行时与编译期。 [1] 

目前，**GCC 15+** 和 **Clang 19+** 已经实现了对该提案的完整支持。您在平时编写模板库或者使用编译期容器时，是否有遇到过受限于 `constexpr` 无法初始化特定类型的情况呢？我们可以围绕您的实际场景进一步聊聊！

[1] [https://www.sandordargo.com](https://www.sandordargo.com/blog/2025/04/23/cpp26-constexpr-language-changes)

[2] [https://www.open-std.org](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2024/p2747r2.html)

[3] [https://isocpp.org](https://isocpp.org/blog/2025/05/cpp26-more-constexpr-in-the-core-language-sandor-dargo)

[4] [https://stackoverflow.com](https://stackoverflow.com/questions/73843218/why-cant-a-placement-new-expression-be-a-constant-expression)

[5] [https://github.com](https://github.com/llvm/llvm-project/issues/107593)

[6] [https://www.open-std.org](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2024/p2747r2.html)