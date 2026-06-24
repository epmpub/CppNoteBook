C++26 std::filesystem::path::display_string() 和 native_encoded_string()

（提案 P2319R5）。这是对 std::filesystem::path 字符串观察器（string observers）的重要补充，主要解决长期以来的编码转换和显示友好性问题，尤其在 Windows 和跨平台代码中。1. 背景问题std::filesystem::path 内部以原生格式存储路径：

- POSIX：char + UTF-8（通常）。
- Windows：wchar_t + UTF-16。

以前的观察器存在以下痛点：

- path::string() 在 Windows 上会使用当前代码页（code page）进行窄转换，容易导致非 ASCII 字符损坏（莫名其妙的乱码）。
- 缺乏一个明确用于显示的字符串版本（适合日志、UI、打印等）。
- 缺乏一个明确使用原生编码的版本，方便直接传给操作系统 API。
- 新增的两个成员函数

cpp

```cpp
namespace std::filesystem {

class path {
public:
    // C++26 新增
    std::string display_string() const;                    // 用于人类可读显示
    std::string native_encoded_string() const;             // 原生编码的窄字符串

    // 对应的 generic 版本（可选）
    std::string generic_display_string() const;
    std::string generic_native_encoded_string() const;
};
}
```

display_string()

- 目的：返回适合显示的 std::string（通常是 UTF-8）。
- 行为：
  - 在 POSIX 上：通常等价于 u8string() 或 string()。
  - 在 Windows 上：从 UTF-16 安全转换为 UTF-8（使用 Unicode 转换，避免当前代码页）。
  - 对无效编码单元使用替换字符（U+FFFD）或其他安全处理。
- 主要用途：日志输出、UI 显示、std::cout、std::format、std::print 等。

native_encoded_string()

- 目的：返回原生编码的 std::string。
- 行为：
  - 在 POSIX 上：通常等价于 string()（UTF-8）。
  - 在 Windows 上：从 UTF-16 转换为当前代码页（或系统窄编码），行为与旧的 string() 类似，但语义更清晰。
- 主要用途：需要传给只接受窄字符串的旧 API，或明确需要原生编码的场景。
- 使用示例

cpp

```cpp
#include <filesystem>
#include <iostream>
#include <format>

int main() {
    std::filesystem::path p = L"C:\\用户\\文档\\文件.txt";  // Windows 示例

    std::cout << "display_string: " << p.display_string() << '\n';           // UTF-8，推荐用于显示
    std::cout << "native_encoded_string: " << p.native_encoded_string() << '\n';

    // 与 std::format 配合（C++26）
    std::cout << std::format("Path: {}\n", p);   // 内部使用 display_string 语义
}
```

4. 与原有方法的对比

| 方法                         | 返回类型      | Windows 行为             | 推荐用途               | C++26 状态         |
| ---------------------------- | ------------- | ------------------------ | ---------------------- | ------------------ |
| string()                     | std::string   | 当前代码页转换（易乱码） | 旧代码                 | 保留（不推荐显示） |
| u8string()                   | std::u8string | UTF-8                    | 明确需要 UTF-8         | 推荐               |
| display_string() (新)        | std::string   | 安全 UTF-8               | 显示、日志、UI（推荐） | 新增               |
| native_encoded_string() (新) | std::string   | 原生窄编码               | 需要原生 API 时        | 新增               |
| wstring()                    | std::wstring  | 原生 UTF-16              | Windows API            | 保留               |

5. 设计要点

- 这两个函数主要服务于编码安全和语义清晰。
- 与 C++26 新增的 std::formatter<std::filesystem::path> 配合良好（默认使用 display_string 风格）。
- 旧的 string() 在 C++26 中被标记为 deprecated（建议改用 display_string 或 u8string）。
- 特性测试宏：__cpp_lib_format_path（与 formatter 共享）。

总结display_string() 和 native_encoded_string() 是 C++26 对 std::filesystem::path 的可用性与安全性改进。强烈推荐在新代码中使用 display_string() 来显示路径，它解决了 Windows 上长期存在的 UTF-8 乱码问题，让路径输出变得可靠且现代。这也是 std::format 对 path 支持的重要基础。需要更多关于 generic_* 版本、与 std::print 的结合示例，还是与旧行为的详细对比吗？