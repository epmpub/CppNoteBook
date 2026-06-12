在 C++26 中，通过核心提案 **[P3068R6](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2024/p3068r6.html)**（由 Hana Dusíková 提出），C++ 标准实现了一项革命性的突破：**正式允许在编译期常量求值（Constant Evaluation）期间抛出（`throw`）并捕获（`catch`）异常**。 [1] 

这一特性消除了传统 `constexpr` 代码与运行时代码之间仅存的重大鸿沟之一，使得“编译期异常处理”成为现实。 [1] 

------

## 1. 历史痛点（C++26 之前）

在 C++26 之前，**`throw` 表达式在编译期核心常量表达式中是绝对禁止的**。
虽然标准允许在 `constexpr` 函数中编写 `throw` 语句，但那只是为了**兼容运行时**。一旦该函数在编译期被触发求值，且代码路径意外走到了 `throw`，编译器就会直接抛出毁灭性的**硬编译错误（Compilation Error）**。 [1, 2] 

## 过去的尴尬折中方案：

因为无法捕获异常，开发者在编写需要在编译期和运行时通用的底层库（如 `std::format`、字符串解析器、JSON 解析器）时，不得不分裂成两套逻辑：

- 在运行时：老老实实 `throw` 异常。
- 在编译期：利用 `if consteval` 或 `std::is_constant_evaluated()` 绕过 `throw`，改用返回 `std::optional`、`std::expected` 或故意触发未定义行为（如 `panic()`）来强行中断编译。

这种不统一使得**复用已有库的常规错误处理逻辑在编译期完全失效**。

------

## 2. C++26 的新特性：编译期异常捕获

C++26 彻底解禁了这一限制。只要异常的**抛出、传递和捕获完全发生在编译期求值的边界内部**，它就是完全合法且能够正常工作的。 [3] 

## 核心代码示例：编译期解析字符串

```cpp
#include <stdexcept>
#include <string_view>

// 一个编译期和运行时通用的日期解析函数
constexpr int parse_year(std::string_view s) {
    if (s.size() != 4) {
        throw std::invalid_argument("Year must be exactly 4 digits"); // C++26 编译期允许抛出！
    }
    int year = 0;
    for (char c : s) {
        if (c < '0' || c > '1') throw std::out_of_range("Invalid year character");
        year = year * 10 + (c - '0');
    }
    return year;
}

// 核心测试
constexpr int test_compile_time_exception() {
    try {
        return parse_year("202a"); // 触发异常
    } catch (const std::out_of_range& e) {
        return 2026; // C++26 编译期可以完美捕获，并纠正返回值
    } catch (...) {
        return -1;
    }
}

// 编译期断言成功！
static_assert(test_compile_time_exception() == 2026);
```

------

## 3. “瞬态”生命周期限制（Transient Restriction）

类似于 C++20 引入的“编译期动态内存分配（`constexpr` allocation）”，C++26 的编译期异常也遵循**严格的生命周期局限性（Transient）**： [1, 4] 

- **严禁逃逸到运行时**：异常对象从抛出到被捕获的整个生命周期，必须**完全闭环在同一个常量求值的全表达式中**。 [3, 4] 
- **未捕获的异常依然是编译错误**：如果你在编译期抛出了异常，但是**没有**任何 `catch` 块去捕获它，或者它一路飞出了 `constexpr` 表达式的边界，程序依然无法编译通过。 [1] 
- **更好的编译诊断**：与老版本直接报错不同，C++26 编译器在遇到这种“未捕获的逃逸异常”时，会利用其附带的错误文本（如 `e.what()`）为你打印出极其直观、精准的**编译期调用栈和错误诊断信息**。 [1] 

------

## 4. 为什么这个特性对 C++26 至关重要？

这一特性的落地，实际上是为了给 C++26 的头号王牌特性 —— **静态反射（Static Reflection - [P2996](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2025/p2996r11.html)）** 铺路。 [1, 5] 

在静态反射中，开发者需要在编译期大批量处理元数据、解析代码、生成新类型、构建复杂的 DSL 语法树。在这种庞大的编译期计算逻辑中，如果没有完善的 `try-catch` 异常恢复机制，任何一个微小的元数据错误都会导致整个项目的编译直接中断，元编程将变得痛苦不堪。 [5, 6] 

有了 `constexpr` 异常，元编程库就能够在编译期平稳地捕获错误，回溯状态，并输出高质量的自定义编译错误提示。

------

## 总结

C++26 的 **Constexpr Exceptions** 实现了 `constexpr` 编程范式的闭环：

1. **代码复用最大化**：运行时和编译期可以无缝共享一套完全相同的基于异常的报错架构。
2. **元编程革命**：为静态反射和大型编译期编译器（如编译期正则表达式解析、编译期 SQL 验证）提供了现代化的局部容错手段。 [1] 

目前，该提案已合入 C++26 国际工作草案，**GCC 15+** (已合并至主干) 和 **Clang 19+** 正在积极推进其标准库与核心语言底层的全套落地。 [5] 

您在目前的开发中，是否有一些复杂的编译期校验（例如在编译期验证配置字符串、URL 格式等），原本需要大费周章地写各种探测代码？我们可以聊聊这个新特性如何帮您简化它！

[1] [https://www.sandordargo.com](https://www.sandordargo.com/blog/2025/05/07/cpp26-constexpr-exceptions)

[2] [https://www.reddit.com](https://www.reddit.com/r/cpp/comments/1gy1x5k/constexpr_exception_throwing_in_c_is_now_in_26/)

[3] [https://www.sandordargo.com](https://www.sandordargo.com/blog/2025/05/07/cpp26-constexpr-exceptions)

[4] [https://www.open-std.org](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2024/p3068r5.html)

[5] [https://news.hada.io](https://news.hada.io/topic?id=28018)

[6] [https://www.modernescpp.com](https://www.modernescpp.com/index.php/reflection-in-c26-determine-the-layout/)