C++26 concatenation of strings and string views 

C++26 新特性：std::string 与 std::string_view 的直接拼接（Concatenation）这是 C++26 中一个重要的易用性改进（提案 P2591R5），解决了从 C++17 引入 std::string_view 以来长期存在的一个痛点。之前的问题（C++23 及更早）你不能直接用 operator+ 拼接 std::string 和 std::string_view：

cpp

```cpp
std::string s = "Hello";
std::string_view sv = " World";

auto result1 = s + sv;           // ❌ 编译错误！C++23 中不存在匹配的 operator+
auto result2 = sv + s;           // ❌ 同样错误
auto result3 = "Hello"s + sv;    // ❌ 也不行
```

以前的尴尬 workaround：

- s + std::string(sv) → 多一次内存分配，不高效
- s += sv; → 可以（因为成员函数支持），但只能用于 +=，不能用于 +
- s.append(sv) 或 s.insert(...)

C++26 的改进新增了多组 operator+ 重载，支持以下所有合法组合（char、const char*、string、string_view 之间）：

cpp

```cpp
std::string s = "Hello";
std::string_view sv = " World";

// C++26 现在全部支持：
auto r1 = s + sv;                // OK
auto r2 = sv + s;                // OK
auto r3 = "Hello"s + sv;         // OK
auto r4 = sv + " World"s;        // OK
auto r5 = s + " literal";        // 原本就支持
auto r6 = sv + " literal";       // 现在也更方便
```

这些重载会返回 std::basic_string（即 std::string 或 std::wstring 等），并进行必要的内存分配。示例

cpp

```cpp
#include <string>
#include <string_view>
#include <iostream>

int main() {
    std::string str = "C++";
    std::string_view view = "26";

    std::string result = str + view + " is awesome!";  // 完全合法

    std::cout << result << '\n';   // C++26 is awesome!
}
```

为什么之前不加？为什么现在加？

- 早期担心引入过多重载会导致歧义或性能意外。
- 长期以来社区反馈强烈，认为这是一个明显的API 不完整（ergonomic gap）。
- += / append() 早已支持 string_view，只是 + 不支持，显得很不一致。
- P2591 最终被采纳，作为 C++26 的纯加法改进（无 breaking change）。

特性测试宏（如果需要条件编译）：

cpp

```cpp
#if __cpp_lib_string_view >= 202403L  // 具体值请以 cppreference 为准
// 支持 string + string_view
#endif
```

总结：
C++26 终于让 std::string 和 std::string_view 可以自然、直观地使用 + 进行拼接，极大提升了日常字符串处理的舒适度。这是典型的“迟到的易用性修复”。需要我给你更多组合示例（包括 wchar_t、用户自定义字符类型等）吗？