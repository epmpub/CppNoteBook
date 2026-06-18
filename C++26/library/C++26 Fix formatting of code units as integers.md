## Fix formatting of code units as integers

C++26 DR20（提案 P2909R4）：Fix formatting of code units as integers（也称“Dude, where’s my char?”）是对 std::format（及相关格式化设施）的一个重要缺陷修复。 

背景与问题在 C++20 引入 std::format 时，字符类型（char、signed char、unsigned char、wchar_t 等）作为code unit（代码单元）被允许用整数格式说明符（如 d、x、X、o 等）格式化。核心问题：

- char 的 signedness 是 implementation-defined（实现定义的）：在很多平台上是 signed。
- std::format 内部通过 std::to_chars 处理整数，而 char 会提升（promote）为 int（始终 signed）。
- 结果：当 char 的值在 signed 范围内为负时，用十六进制/八进制等格式输出时会出现负号或不符合预期的值（如 0xFF 可能输出成负数形式）。 sandordargo.com

示例（C++23 及之前的行为，可能因平台不同而异）：

cpp

```cpp
for (char c : std::string("🤷")) {  // UTF-8 多字节
    std::print("\\x{:02x}", c);
}
```

- 可能输出 \xf0\x9f\xa4\xb7（正确，无符号行为）
- 也可能输出 \x-10\x-61\x-5c\x-49（带负号的错误输出）

用户期望代码单元总是以无符号整数形式输出（尤其是调试 UTF-8/UTF-16 等文本时），但实际行为依赖平台且常令人惊讶。解决方案（C++26 的修复）

- 当使用整数格式说明符格式化字符类型（code unit types）时，总是先转换为对应的无符号类型（unsigned char、unsigned wchar_t 等）再进行格式化。
- 这保证了输出始终是无符号的、符合直觉的十六进制/十进制等表示。
- 引入了新的特性测试宏：__cpp_lib_format_uchar（202311L）。 en.cppreference.com

效果：

- 跨平台行为一致。
- char 作为代码单元时的格式化更符合文本处理预期。
- 对 char 作为普通字符打印（{:c}）或字符串格式化无影响。

实际影响

- 这是一个 breaking change，但作为 Defect Report (DR) 处理，已回溯应用到 C++23（以及部分 C++20）。
- 三大编译器已全部实现：GCC 14+、Clang 18+、MSVC 19.40+。
- 大多数现有代码不受影响；受影响的代码主要是依赖特定 signed 输出行为的调试/低级代码，现在会得到更一致、无符号的结果。

一句话总结：这个 DR 修复了 std::format 对 char 等代码单元用整数格式输出时的平台不一致和不符合直觉的问题，强制转为无符号整数格式，使输出稳定且合理（尤其适合处理 UTF-8 等多字节序列）。这是 std::format 安全性和可用性改进系列中的一环。 

