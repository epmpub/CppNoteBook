

# std::from_chars 将字符串转换为整数和浮点数

代码:

```C++
#include <string_view>
#include <charconv>
#include <print>

int main() {
    std::string_view i1 = "1234";
    int o1{};
    std::from_chars(i1.data(), i1.data()+i1.size(), o1);
    // o1 == 1234

    std::println("o1 == {}\n", o1);

    // The function returns information about potential errors
    // and the end of parsed section
    std::string_view i2 = "123Hello";
    int o2{};
    {
    auto [ptr, err] = std::from_chars(i2.data(), i2.data()+i2.size(), o2);
    // *ptr == 'H', err == Success
    std::println("*ptr == {}, err == {}", *ptr, std::make_error_condition(err).message());
    std::println("o2 == {}", o2);
    }

    std::string_view i3 = "-4";
    unsigned o3{};
    {
    auto [ptr, err] = std::from_chars(i3.data(), i3.data()+i3.size(), o3);
    // *ptr == '-', err == Invalid argument
    std::println("*ptr == {}, err == {}", *ptr, std::make_error_condition(err).message());
    }

    int o4{};
    {
    auto [ptr, err] = std::from_chars(i3.data(), i3.data()+i3.size(), o4);
    // ptr == i3.end(), err == Success
    std::println("(ptr == i3.end) == {}, err == {}", (ptr == i3.end()), std::make_error_condition(err).message());
    std::println("o4 == {}\n", o4);
    }

    // Base can be specified (but the function doesn't parse the prefix)
    std::string_view i4 = "0xFE"; // hexadecimal
    std::string_view i5 = "077";  // octal
    int o5{};
    {
    i4 = i4.substr(2); // Skip the hexadecimal prefix
    std::from_chars(i4.data(), i4.data()+i4.size(), o5, 16);
    // o5 == 0xfe
    std::println("o5 == {} / 0x{:x}", o5, o5);

    i5 = i5.substr(1); // Skip the octal prefix
    std::from_chars(i5.data(), i5.data()+i5.size(), o5, 8);
    // o5 == 077
    std::println("o5 == {} / 0{:o}\n", o5, o5);
    }

    // Floating point support
    std::string_view i6 = "3.14";
    double o6{};
    std::from_chars(i6.data(), i6.data()+i6.size(), o6);
    std::println("o6 == {}", o6);

    std::string_view i7 = "0.1e-15";
    double o7{};
    std::from_chars(i7.data(), i7.data()+i7.size(), o7);
    std::println("o7 == {}", o7);
}
```

这段代码展示了 C++17 引入的 <charconv> 头文件中的 std::from_chars 函数，用于将字符串转换为数值类型（如整数和浮点数）。它通过示例展示了基本用法、错误处理、进制指定和浮点数支持。以下是逐步解释。

------

代码概览

- 使用 std::from_chars 将字符串视图转换为 int、unsigned 和 double。
- 处理返回的指针和错误码，验证解析结果。
- 支持不同进制和浮点数格式。

------

关键组件

1. **头文件**

cpp

```cpp
#include <string_view>
#include <charconv>
#include <print>
```

- <string_view>：提供 std::string_view。
- <charconv>：提供 std::from_chars。
- <print>：C++23 的 std::println 用于格式化输出。
- **基本整数转换**

cpp

```cpp
std::string_view i1 = "1234";
int o1{};
std::from_chars(i1.data(), i1.data()+i1.size(), o1);
std::println("o1 == {}\n", o1);
```

- **std::from_chars**：

  - 原型：from_chars(first, last, value)。
  - 从 [first, last) 解析整数，存储到 value。

- **调用**：

  - i1 = "1234"，解析为 o1 = 1234。

- **输出**：

  ```text
  o1 == 1234
  ```

- **错误处理和部分解析**

cpp

```cpp
std::string_view i2 = "123Hello";
int o2{};
auto [ptr, err] = std::from_chars(i2.data(), i2.data()+i2.size(), o2);
std::println("*ptr == {}, err == {}", *ptr, std::make_error_condition(err).message());
std::println("o2 == {}", o2);
```

- **返回类型**：

  - std::from_chars_result { char* ptr; std::errc err; }。
  - ptr：停止解析的位置。
  - err：错误码（std::errc::success 或其他）。

- **调用**：

  - i2 = "123Hello"，解析 123，停止于 'H'。
  - o2 = 123, ptr 指向 'H', err = success。

- **输出**：

  ```text
  *ptr == H, err == Success
  o2 == 123
  ```

- **无效输入**

cpp

```cpp
std::string_view i3 = "-4";
unsigned o3{};
auto [ptr, err] = std::from_chars(i3.data(), i3.data()+i3.size(), o3);
std::println("*ptr == {}, err == {}", *ptr, std::make_error_condition(err).message());
```

- **调用**：

  - i3 = "-4"，尝试解析为 unsigned。
  - 负数对无符号类型无效，解析失败。
  - ptr 指向 '-', err = invalid_argument。

- **输出**：

  ```text
  *ptr == -, err == Invalid argument
  ```

- **负数支持**

cpp

```cpp
int o4{};
auto [ptr, err] = std::from_chars(i3.data(), i3.data()+i3.size(), o4);
std::println("(ptr == i3.end) == {}, err == {}", (ptr == i3.end()), std::make_error_condition(err).message());
std::println("o4 == {}\n", o4);
```

- **调用**：

  - i3 = "-4"，解析为有符号 int。
  - o4 = -4, ptr 指向末尾，err = success。

- **输出**：

  ```text
  (ptr == i3.end) == true, err == Success
  o4 == -4
  ```

- **指定进制**

cpp

```cpp
std::string_view i4 = "0xFE";
std::string_view i5 = "077";
int o5{};
i4 = i4.substr(2);
std::from_chars(i4.data(), i4.data()+i4.size(), o5, 16);
std::println("o5 == {} / 0x{:x}", o5, o5);
i5 = i5.substr(1);
std::from_chars(i5.data(), i5.data()+i5.size(), o5, 8);
std::println("o5 == {} / 0{:o}\n", o5, o5);
```

- **原型扩展**：

  - from_chars(first, last, value, base)。
  - base 指定进制（2-36）。

- **i4 = "0xFE"**：

  - 跳过 "0x"，解析 "FE"（16 进制）。
  - o5 = 254（0xFE）。

- **i5 = "077"**：

  - 跳过 "0"，解析 "77"（8 进制）。
  - o5 = 63（077）。

- **输出**：

  ```text
  o5 == 254 / 0xfe
  o5 == 63 / 077
  ```

- **浮点数支持**

cpp

```cpp
std::string_view i6 = "3.14";
double o6{};
std::from_chars(i6.data(), i6.data()+i6.size(), o6);
std::println("o6 == {}", o6);
std::string_view i7 = "0.1e-15";
double o7{};
std::from_chars(i7.data(), i7.data()+i7.size(), o7);
std::println("o7 == {}", o7);
```

- **调用**：

  - i6 = "3.14"，解析为 o6 = 3.14。
  - i7 = "0.1e-15"，解析为 o7 = 0.1 × 10⁻¹⁵ = 1e-16。

- **输出**：

  ```text
  o6 == 3.14
  o7 == 1e-16
  ```

------

为什么这样工作？

1. **std::from_chars**：
   - 高效解析字符串到数值，无动态内存分配。
   - 返回指针和错误码，便于错误处理。
2. **进制支持**：
   - 手动跳过前缀（如 "0x"），指定基数解析。
3. **浮点数**：
   - 支持十进制和科学计数法。

------

输出

```text
o1 == 1234

*ptr == H, err == Success
o2 == 123

*ptr == -, err == Invalid argument

(ptr == i3.end) == true, err == Success
o4 == -4

o5 == 254 / 0xfe
o5 == 63 / 077

o6 == 3.14
o7 == 1e-16
```

------

使用场景

- **性能敏感场景**：
  - 比 std::stoi 或 std::stringstream 更快。
- **错误处理**：
  - 检查解析失败或部分解析。
- **多进制解析**：
  - 处理不同格式的输入。

------

总结

- std::from_chars 将字符串转换为整数和浮点数。
- 返回指针和错误码，支持错误检测。
- 可指定进制，处理科学计数法。
- 代码展示了其高效性和灵活性。
- 