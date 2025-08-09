

# C++  std::to_chars 函数，用于将数值（整数或浮点数）高效地格式化为字符序列

```C++
#include <charconv>
#include <string_view>
#include <numbers>

char buffer[5];
{
// Formatting integers
auto [end, err] = std::to_chars(buffer, buffer+5, 12345);
// [buffer, end) == "12345"
}

{
// With a custom base
auto [end, err] = std::to_chars(buffer, buffer+5, 12345, 35);
// [buffer, end) == "a2p"
}

{
// If the formatted values doesn't fit the buffer
// error is returned and the buffer is left in unspecified state
auto [end, err] = std::to_chars(buffer, buffer+5, 12345, 2);
// std::make_error_code(err).message() == 
//   "Value too large for defined data type"
}

{
// Formatting floating-point
auto [end, err] = std::to_chars(buffer, buffer+5, 3.14);
// [buffer, end) == "3.14"
}

{
// With custom format and precision
auto [end, err] = std::to_chars(buffer, buffer+5, std::numbers::pi,
                                std::chars_format::fixed, 3);
// [buffer, end) == "3.142"
}

// Different floating point formats
char buf[1024];
{
auto [end, _] = std::to_chars(buf, buf+1024, std::numbers::pi * 1000,
                              std::chars_format::scientific);
// [buf, end) == 3.141592653589793e+03
}

{
auto [end, _] = std::to_chars(buf, buf+1024, std::numbers::pi * 1000,
                              std::chars_format::fixed);
// [buf, end) == 3141.592653589793
}

{
auto [end, _] = std::to_chars(buf, buf+1024, std::numbers::pi * 1000,
                              std::chars_format::hex);
// [buf, end) == 1.88b2f704a9409p+11
}

{
// General switches between fixed and specientific
auto [end, _] = std::to_chars(buf, buf+1024, std::numbers::pi * 1000,
                              std::chars_format::general);
// [buf, end) == 3141.592653589793
}

{
auto [end, _] = std::to_chars(buf, buf+1024, std::numbers::pi * 1000,
                              std::chars_format::general, 3);
// [buf, end) == 3.14e+03
}
```

这段代码展示了 C++17 中引入的 <charconv> 库的 std::to_chars 函数，用于将数值（整数或浮点数）高效地格式化为字符序列。它比传统的 sprintf 或 std::stringstream 更快，因为它避免了 locale 和动态内存分配。代码通过多个示例展示了整数和浮点数的格式化，包括自定义进制、浮点格式和精度控制。以下是对代码的详细解释：

------

**代码结构**

1. **头文件**：
   - <charconv>：提供 std::to_chars 和 std::chars_format。
   - <string_view>：未直接使用，但常与 <charconv> 搭配。
   - <numbers>：C++20 提供数学常数（如 std::numbers::pi）。
2. **主要内容**：
   - 整数格式化（十进制和其他进制）。
   - 浮点数格式化（默认、固定、科学、十六进制、一般格式）。
   - 处理缓冲区溢出和错误。

------

**代码逐步解释**

**1. 整数格式化（十进制）**

cpp

```cpp
char buffer[5];
auto [end, err] = std::to_chars(buffer, buffer+5, 12345);
// [buffer, end) == "12345"
```

- **std::to_chars**：
  - 将整数 12345 写入 buffer。
  - 参数：起始指针、结束指针、值。
  - 返回 std::to_chars_result：{end, err}。
    - end：写入结束位置。
    - err：错误码（std::errc）。
- **结果**：
  - buffer 包含 "12345"，end == buffer + 5，err == std::errc{}（无错误）。
  - 缓冲区恰好够用（5 字节）。

------

**2. 整数格式化（自定义进制）**

cpp

```cpp
auto [end, err] = std::to_chars(buffer, buffer+5, 12345, 35);
// [buffer, end) == "a2p"
```

- **参数**：
  - 最后一个参数 35 指定进制（2-36）。
  - 12345 在 35 进制下：
    - 12345 = 10*35^2 + 2*35 + 25。
    - 10 = 'a', 2 = '2', 25 = 'p'。
- **结果**：
  - buffer 包含 "a2p"，end == buffer + 3，err == std::errc{}。

------

**3. 缓冲区不足**

cpp

```cpp
auto [end, err] = std::to_chars(buffer, buffer+5, 12345, 2);
// std::make_error_code(err).message() == "Value too large for defined data type"
```

- **参数**：
  - 进制 2，12345 的二进制表示为 "11000000111001"（14 位）。
  - 缓冲区只有 5 字节，不足以存储。
- **结果**：
  - err == std::errc::value_too_large。
  - buffer 内容未定义，end 未指定。

------

**4. 浮点数格式化（默认）**

cpp

```cpp
auto [end, err] = std::to_chars(buffer, buffer+5, 3.14);
// [buffer, end) == "3.14"
```

- **默认格式**：
  - 无精度或格式参数时，使用最短表示。
- **结果**：
  - buffer 包含 "3.14"，end == buffer + 4，err == std::errc{}。

------

**5. 浮点数格式化（固定格式，指定精度）**

cpp

```cpp
auto [end, err] = std::to_chars(buffer, buffer+5, std::numbers::pi,
                                std::chars_format::fixed, 3);
// [buffer, end) == "3.142"
```

- **参数**：
  - std::numbers::pi ≈ 3.1415926535。
  - std::chars_format::fixed：固定小数点格式。
  - 3：精度（小数点后 3 位，四舍五入）。
- **结果**：
  - buffer 包含 "3.142"，end == buffer + 5，err == std::errc{}。

------

**6. 不同浮点格式**

cpp

```cpp
char buf[1024];
{
auto [end, _] = std::to_chars(buf, buf+1024, std::numbers::pi * 1000,
                              std::chars_format::scientific);
// [buf, end) == "3.141592653589793e+03"
}
```

- **scientific**：
  - 科学计数法，π * 1000 ≈ 3141.592653589793。
  - 输出 "3.141592653589793e+03"。

cpp

```cpp
{
auto [end, _] = std::to_chars(buf, buf+1024, std::numbers::pi * 1000,
                              std::chars_format::fixed);
// [buf, end) == "3141.592653589793"
}
```

- **fixed**：
  - 固定小数点，默认精度为 6（或实现定义）。
  - 输出 "3141.592653589793"。

cpp

```cpp
{
auto [end, _] = std::to_chars(buf, buf+1024, std::numbers::pi * 1000,
                              std::chars_format::hex);
// [buf, end) == "1.88b2f704a9409p+11"
}
```

- **hex**：
  - 十六进制浮点表示。
  - 3141.592653589793 ≈ 1.88b2f704a9409 * 2^11。
  - 输出 "1.88b2f704a9409p+11"。

------

**7. 一般格式（general）**

cpp

```cpp
{
auto [end, _] = std::to_chars(buf, buf+1024, std::numbers::pi * 1000,
                              std::chars_format::general);
// [buf, end) == "3141.592653589793"
}
```

- **general**：
  - 根据值选择 fixed 或 scientific，优先简洁。
  - 这里选择 fixed，输出 "3141.592653589793"。

cpp

```cpp
{
auto [end, _] = std::to_chars(buf, buf+1024, std::numbers::pi * 1000,
                              std::chars_format::general, 3);
// [buf, end) == "3.14e+03"
}
```

- **带精度**：
  - 精度 3，选择 scientific 表示更简洁。
  - 输出 "3.14e+03"。

------

**关键技术点**

1. **std::to_chars**：
   - 高效地将数值转换为字符，无动态分配。
   - 返回结构化绑定 {end, err}。
2. **进制支持**：
   - 整数支持 2-36 进制。
3. **浮点格式**：
   - fixed：固定小数点。
   - scientific：科学计数法。
   - hex：十六进制浮点。
   - general：自动选择。
4. **错误处理**：
   - 缓冲区不足返回 std::errc::value_too_large。
5. **<numbers>**：
   - 提供数学常数，如 pi。

------

**可能的改进或注意事项**

1. **输出验证**：
   - 当前无 main，可添加打印检查结果。
2. **缓冲区大小**：
   - buffer[5] 易溢出，建议动态计算。
3. **错误检查**：
   - 可显式处理 err。

------

**总结**

- **功能**：展示 std::to_chars 的整数和浮点格式化。
- **优势**：高效、灵活、无 locale 依赖。
- **用途**：日志、序列化等场景。

如果你有具体问题（例如如何解析回数值），欢迎提问！