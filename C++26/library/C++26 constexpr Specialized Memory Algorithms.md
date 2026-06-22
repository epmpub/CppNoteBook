##### C++26 `constexpr` Specialized Memory Algorithms

```c
#include <array>
#include <algorithm>
#include <iostream>

constexpr auto make_array()
{
    std::array<int,5> arr{};

    constexpr std::array src{1,2,3,4,5};

    std::copy(src.begin(), src.end(), arr.begin());

    return arr;
}

int main()
{
    constexpr auto result = make_array();

    for (int x : result)
        std::cout << x << ' ';
}

//因为 std::copy 才是对已构造对象进行赋值，而 std::uninitialized_copy 是对未构造对象进行构造。两者用途完全不同。
```

**C++26 `constexpr` Specialized Memory Algorithms** 是通过提案 **P3508R0**（以及早期 **P2283R2**）引入的特性，主要针对 `<memory>` 头文件中的**未初始化内存算法**（Uninitialized Memory Algorithms）。

### 背景
C++20 已将大量标准算法标记为 `constexpr`，并支持 `constexpr` 容器（如 `std::vector`、`std::string`），但**未初始化内存相关算法**被遗漏了。这些算法对于实现 `constexpr` 容器（尤其是自定义容器或需要手动管理未初始化内存的场景）至关重要。

C++26 补上了这个缺口，让这些底层内存操作也能在**常量求值**（constant evaluation）中使用。

### 受影响的算法（`std` 和 `std::ranges` 命名空间）

以下函数及其 `_n` / `_result` 变体现在都是 `constexpr`：

- `uninitialized_value_construct` / `uninitialized_value_construct_n`
- `uninitialized_copy` / `uninitialized_copy_n` / `uninitialized_copy_result`
- `uninitialized_move` / `uninitialized_move_n` / `uninitialized_move_result`
- `uninitialized_fill` / `uninitialized_fill_n`
- `uninitialized_default_construct` / `uninitialized_default_construct_n`（通过 P3369R0 补充）

这些算法的行为与之前一致，但在 `constexpr` 上下文中合法。

**Feature Test Macro**（C++26）：
```cpp
#if __cpp_lib_raw_memory_algorithms >= 202411L  // 或类似值
// 支持 constexpr specialized memory algorithms
#endif
```

### 示例

```cpp
#include <memory>
#include <array>

constexpr auto make_array() {
    std::array<int, 5> arr{};
    auto* p = arr.data();
    
    // 在 constexpr 中安全使用
    std::uninitialized_value_construct(p, p + 5);  // C++26 前错误
    // 或
    std::uninitialized_copy(std::array{1,2,3,4,5}.begin(), 
                            std::array{1,2,3,4,5}.end(), p);
    
    return arr;
}

constexpr auto result = make_array();
```

在常量求值中，这些算法使用 `std::construct_at`（已为 `constexpr`）或 `constexpr` placement new（P2747）来构造对象。

### 为什么重要？

- **支持 `constexpr` 容器实现**：自定义 `constexpr` vector-like 容器现在可以更完整地在编译时分配/构造元素，而无需绕过标准算法。
- **代码复用**：同一份容器或内存管理代码可在编译时（测试、初始化常量数据）和运行时使用。
- **编译时计算增强**：便于在 `constexpr` 函数中构建复杂数据结构（如查找表、静态资源等）。

### 注意事项

1. **构造与析构**：在 `constexpr` 中必须严格遵守对象生命周期（构造后必须析构，否则常量求值失败）。
2. **瞬态分配**（Transient Allocation）：C++20+ 已支持 `constexpr` `new`/`delete`，但分配的内存必须在同一常量求值中释放。
3. **实现**：标准库实现主要只需更新函数签名（因为底层 `construct_at` 已支持 `constexpr`），部分实现（如 GCC）已合并相关 patch。
4. **与 placement new**：P2747R2 让 placement new 本身也成为 `constexpr`，进一步简化了这些算法的实现。

### 总结

这是 C++26 “constexpr 大扩张”的一部分，让标准库对编译时编程的支持更加完整和实用。特别有利于库作者和需要编译时构建复杂数据的开发者。

更多细节可参考：
- [P3508R0 Wording](https://wg21.link/P3508R0)
- cppreference: [Uninitialized memory algorithms](https://en.cppreference.com/w/cpp/memory)（C++26 部分已标记）

如果需要具体某个算法的详细示例或与其他 constexpr 特性的对比，请随时补充！