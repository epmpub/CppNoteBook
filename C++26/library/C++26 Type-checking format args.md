### Type-checking format args

在 **C++26** 中，通过了关键提案 [P2757R3: Type-checking format args](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2023/p2757r3.html)，对 `std::format` 的编译期检查能力进行了极为重要的补全：**现在，动态格式化参数（如通过 `{}` 传递的动态宽度或动态精度）的类型错误，也能在编译期被彻底拦截，而不再是在运行期抛出异常**。 [1] 

这项改进消除了一类长期隐藏的 `std::format` 运行期崩溃隐患。 [1] 

------

## 🔍 过去 C++20/C++23 的隐患与痛点

虽然 C++20 引入 `std::format` 时就支持“编译期格式字符串检查”（通过 `std::format_string`），但这种检查有一个巨大的**盲区**：**它无法感知嵌套 placeholder（动态宽度/精度）对应的参数类型**。 [2, 3] 

## 导致运行期崩溃的典型代码：

```cpp
// ❌ 在 C++20 / C++23 中：可以通过编译，但运行期抛出 std::format_error 崩溃！
// 🚀 在 C++26 中：直接编译失败（Compile Error）！
std::string s = std::format("{:>{}}", "hello", "10"); 
```

**为什么过去会漏拦截？**

1. 格式字符串语法规定：第一个 `{}` 需要右对齐，并且它的宽度是由第二个 `{}` 动态指定的。
2. 按照标准，动态宽度必须是一个**整数类型**（如 `int`, `size_t`），但这里误传入了字符串 `"10"`。
3. 在 C++26 之前，负责编译期解析的 `basic_format_parse_context` 仅仅知道“这里需要下一个参数”，但**完全不知道下一个参数是什么类型**。这导致编译器以为它是个整数进而放行，最终将错误推迟到了运行期。 [1, 3, 4] 

------

## 🛠️ C++26 的解决方案

C++26 扩展了底层 `std::basic_format_parse_context` 的内部实现，使其在编译期解析格式字符串时，能够**直接访问并擦除式检查后续参数的类型（Type-erased information）**。 [1] 

当你编写自定义类型或调用 `std::format` 时，一旦发生如下错误，都会在**编译时**产生硬错误（Hard Compile Error）：

```cpp
// 1. 动态宽度传入了非整数类型
std::format("{:>{}}", "text", "8");     // ❌ C++26 编译报错：宽度必须是整数，而不是 const char*

// 2. 动态精度传入了非整数类型
std::format("{:.{}}", 3.14159, "2");    // ❌ C++26 编译报错：精度必须是整数

// 3. 混合使用时类型不匹配
std::format("{:{}..{}}", 123, "10", 2); // ❌ C++26 编译报错
```

------

## 🧱 如何在自定义 Formatter 中利用此特性？

如果你在为自己的类编写 `std::formatter`，C++26 提供的新机制并不会破坏你原有的 `parse` 函数签名，而是标准库自动在你的 `parse_context` 中增强了类型校验逻辑。

```cpp
#include <format>

struct MyType {};

template <>
struct std::formatter<MyType> {
    constexpr auto parse(std::format_parse_context& ctx) {
        auto it = ctx.begin();
        if (it != ctx.end() && *it == '}') return it;

        // 如果你使用了嵌套的动态参数：
        if (*it == '{') {
            // C++26 内部的 next_arg_id() / check_arg_id() 
            // 会自动根据当前正在格式化的参数列表校验其类型是否合法
        }
        return it;
    }

    auto format(const MyType&, std::format_context& ctx) const {
        return std::format_to(ctx.out(), "[MyType]");
    }
};
```

## ⚙️ 编译器支持情况与测试宏

你可以通过检查特征测试宏（Feature-test macro）来确认当前编译环境是否开启了此功能： [5] 

```cpp
#include <format>

#if defined(__cpp_lib_format) && __cpp_lib_format >= 202306L
    // C++26 强类型编译期参数检查已启用
#endif
```

- **编译器支持**：
  - **GCC 15+** 已率先完整实现此提案。
  - **Clang / MSVC** 正在对 C++26 的 `<format>` 库进行这一轮补全。 [1, 6, 7] 

如果你有具体的 `std::format` 代码在旧环境中报错，或者想了解 C++26 另一个关于格式化的重要提案 **`std::dynamic_format` (用于运行期动态格式化字符串)**，请随时告诉我！ [8] 

[1] [https://www.sandordargo.com](https://www.sandordargo.com/blog/2025/07/09/cpp26-format-part-1)

[2] [https://cppreference.com](https://cppreference.com/cpp/utility/format/format)

[3] [https://stackoverflow.com](https://stackoverflow.com/questions/77970648/why-does-stdformat-throw-at-runtime-for-incorrect-format-specifiers)

[4] [https://www.open-std.org](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2023/p2757r1.html)

[5] [https://cppreference.com](https://cppreference.com/cpp/26)

[6] [https://cppreference-45864d.gitlab-pages.liu.se](https://cppreference-45864d.gitlab-pages.liu.se/en/cpp/compiler_support/26.html)

[7] [https://github.com](https://github.com/llvm/llvm-project/issues/105378)

[8] [https://www.sandordargo.com](https://www.sandordargo.com/blog/2025/07/16/cpp26-format-part-2)