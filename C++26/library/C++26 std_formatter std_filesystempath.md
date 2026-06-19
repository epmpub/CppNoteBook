## C++26 std::formatter<<std::filesystem::path>>

C++26 中 std::formatter<std::filesystem::path> 支持使用 std::format、std::print 等直接格式化 std::filesystem::path 对象，这是长期缺失的便利特性。 

```cpp
#include <filesystem>
#include <format>

template<class CharT>
struct formatter<std::filesystem::path, CharT>;   // since C++26
```

- 头文件：<filesystem>
- 特性测试宏：__cpp_lib_format_path（202403L）
- 这是一个 debug-enabled 的 formatter（支持 :? 调试格式）。
- 格式说明符（path-format-spec）

text

```text
fill-and-align（可选） width（可选） ?（可选） g（可选）
```

- ?：启用调试/转义格式（escaped string），类似字符串的 :?，会添加引号并转义控制字符（如 \n → \\n）。
- g：使用 generic-format 表示（跨平台 / 分隔符），而不是 native format。
- fill、align、width：与标准格式说明符含义相同（填充、对齐、宽度）。
- 默认行为 vs 调试格式

cpp

```cpp
auto p = std::filesystem::path("/usr/bin");
std::cout << std::format("{}", p);        // 输出: /usr/bin     （不带引号）

auto p2 = std::filesystem::path("multi\nline");
std::cout << std::format("{}", p2);       // 输出: multi
                                          //       line          （不带引号，多行直接输出）

std::cout << std::format("{:?}", p2);     // 输出: "multi\nline" （带引号 + 转义）
```

- 默认（{}）：不加引号，直接输出路径内容（适合普通路径显示）。
- 调试模式（{?} 或调用 set_debug_format()）：添加引号 + 转义控制字符，适合日志、调试等需要精确表示的场景。 sandordargo.com
- g 选项（generic format）

cpp

```cpp
std::filesystem::path p = "C:\\Windows\\System32";
std::format("{}", p);      // Windows 下可能是 C:\Windows\System32（native）
std::format("{:g}", p);    // 强制 generic: C:/Windows/System32
```

5. 字符编码处理

- 当输出 char 而路径是 wchar_t（Windows 常见）且 literal encoding 为 UTF-8 时，会进行 transcoding（UTF-16 → UTF-8），替换非法序列为 U+FFFD。
- 其他情况根据 path::value_type 与 CharT 是否匹配决定是否转码。
- 使用示例

cpp

```cpp
#include <filesystem>
#include <format>
#include <print>

int main() {
    std::filesystem::path p = "/home/user/file.txt";
    
    std::print("Path: {}\n", p);
    std::print("Debug: {:?}\n", p);
    std::print("Generic: {:g}\n", p);
    
    // 结合其他格式
    std::print("{:>40}\n", p);   // 右对齐，宽度40
}
```

7. 实现状态

截至 2026 年初，主要编译器对该特性的支持仍在进行中（可能尚未完全可用）。可以先用第三方库 fmt（#include <fmt/std.h>）体验类似功能。 

sandordargo.com

总结C++26 的 std::formatter<std::filesystem::path> 让路径格式化变得自然且强大：

- 默认行为符合大多数显示需求（不强制加引号）。
- :? 提供安全的调试输出。
- 支持 g 强制 generic 格式。
- 正确处理 Unicode 和平台差异。

这极大提升了日志记录、错误报告和调试体验，不再需要手动调用 .string() 或 .generic_string()。