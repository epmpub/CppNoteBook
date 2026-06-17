**P2198R7《Freestanding Feature-Test Macros and Implementation-Defined Extensions》** 本身并不是“增加更多库功能”的提案，而是为 C++26 的一系列 Freestanding 扩展（P2013、P2338、P2407、P2833 等）建立配套机制。它解决的核心问题是：**如何可靠地检测一个 Freestanding 实现到底提供了哪些标准库设施。** ([Open Standards](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2023/p2198r7.html?utm_source=chatgpt.com))

在 C++20/C++23 中有一个尴尬问题：

```cpp
#ifdef __cpp_lib_filesystem
```

即使 Freestanding 实现根本没有 `<filesystem>`，标准仍要求很多 feature-test macro 被定义，因此这些宏在 Freestanding 环境里可能“撒谎”。P2198R7 修复了这一点。([Open Standards](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2023/p2198r7.html?utm_source=chatgpt.com))

例如：

```cpp
__cpp_lib_filesystem
__cpp_lib_chrono
__cpp_lib_format
```

在 Freestanding 中不再必须定义。

而新增：

```cpp
__cpp_lib_freestanding_feature_test_macros
```

用于表明：

```text
这些 feature-test macros 现在是可信的
```

即：

```cpp
#if defined(__cpp_lib_freestanding_feature_test_macros)
```

意味着你可以相信当前实现报告的 Freestanding 能力。([Open Standards](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2023/p2198r7.html?utm_source=chatgpt.com))

------

P2198R7 还新增了一组按头文件划分的 Freestanding 宏：

```cpp
__cpp_lib_freestanding_utility
__cpp_lib_freestanding_tuple
__cpp_lib_freestanding_ratio
__cpp_lib_freestanding_memory
__cpp_lib_freestanding_functional
__cpp_lib_freestanding_iterator
__cpp_lib_freestanding_ranges
```

对应：

```cpp
<utility>
<tuple>
<ratio>
<memory>
<functional>
<iterator>
<ranges>
```

([Open Standards](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2023/p2198r7.html?utm_source=chatgpt.com))

例如：

```cpp
#if defined(__cpp_lib_freestanding_ranges)
```

即可判断当前 Freestanding 实现是否提供标准要求的 ranges 子集。

------

另一个重要变化是：

```cpp
__cpp_lib_freestanding_operator_new
```

配合 P2013R5（Optional `::operator new`）。

```cpp
__cpp_lib_freestanding_operator_new == 0
```

表示：

```text
当前 Freestanding 环境没有默认全局 operator new
```

例如裸机环境：

```cpp
-freestanding
-nostdlib
```

可能得到：

```cpp
__cpp_lib_freestanding_operator_new == 0
```

而带 heap 的 RTOS 环境可能得到：

```cpp
__cpp_lib_freestanding_operator_new >= 202306L
```

([Open Standards](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2023/p2198r7.html?utm_source=chatgpt.com))

------

P2198R7 还正式确认：

> Freestanding 实现可以提供超出标准最低要求的额外库功能，只要这些功能满足 Hosted 版本的语义要求。([Open Standards](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2023/p2198r7.html?utm_source=chatgpt.com))

例如某个 MCU 厂商完全可以提供：

```cpp
<cmath>
<complex>
<span>
<expected>
```

即使标准没有强制要求。

但如果提供：

```cpp
std::optional
```

则必须是完整的：

```cpp
optional::value()
optional::has_value()
optional::operator*
```

不能只实现半个类。([Open Standards](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2023/p2198r7.html?utm_source=chatgpt.com))

------

P2198R7 间接推动了后续大量 Freestanding 扩展。到 C++26 为止，Freestanding 已经从最初的：

```cpp
<cstdint>
<type_traits>
<new>
```

扩展到包含大量现代库设施：

```cpp
<utility>
<tuple>
<functional>
<memory>
<ranges>
<charconv>
<cstring>
<cstdlib>
<cwchar>
<expected>
<span>
<mdspan>
```

等内容。P2198R7 提供的正是这些能力的统一检测机制。([Open Standards](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2023/p2198r7.html?utm_source=chatgpt.com))

对于实际开发者来说，最有价值的是下面这种写法：

```cpp
#if defined(__cpp_lib_freestanding_ranges)

    auto v =
        std::views::iota(0, 10)
      | std::views::filter([](int x)
            { return x % 2 == 0; });

#else

    // fallback

#endif
```

以前 Freestanding 环境下很难可靠判断是否能用 Ranges；P2198R7 之后，这类检测成为标准化机制。([Open Standards](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2023/p2198r7.html?utm_source=chatgpt.com))

因此可以把 P2198R7 理解为：

```text
P2013 解决 operator new
P2338 扩展 charconv/cstring
P2407 扩展 compare/span
P2833 扩展 expected/mdspan

而 P2198 负责告诉你：
“这些东西到底有没有。”
```

它是整个 C++26 Freestanding 体系的“能力发现（feature discovery）基础设施”。 ([Open Standards](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2023/p2198r7.html?utm_source=chatgpt.com))