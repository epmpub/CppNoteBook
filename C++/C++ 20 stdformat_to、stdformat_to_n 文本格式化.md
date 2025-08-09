

# C++ 20 std::format_to、std::format_to_n 文本格式化

```C++
#include <format>
#include <vector>
#include <string_view>
#include <iostream>
#include <iterator>

int main() {
    std::vector<char> buffer;
    int x = 42;
    // formatting into a dynamic buffer
    std::format_to(std::back_inserter(buffer), "x == {}", x);
    
    std::string_view str1(buffer.begin(), buffer.end());
    // str1 == "x == 42"
    std::cout << str1 << "\n";

    // Also works for output to streams
    std::format_to(std::ostream_iterator<char>(std::cout), "x == {}\n", x);
    // prints: x == 42

    std::array<char, 32> static_buffer;
    // formatting into a static buffer
    std::format_to_n(static_buffer.begin(), 32,
        "Today is {}, Expected temperature is {} Celsius",
        "Tuesday", 24);
    
    std::string_view str2(static_buffer.begin(), static_buffer.end());
    // str2 == "Today is Tuesday, Expected tempe"
    std::cout << str2 << "\n";
}
```

这段代码展示了 C++20 中 <format> 头文件提供的格式化功能，包括 std::format_to 和 std::format_to_n，用于将格式化字符串输出到动态缓冲区、流和静态缓冲区。以下是逐步解释。

------

代码概览

- 使用 std::format_to 将格式化字符串写入动态向量和输出流。
- 使用 std::format_to_n 将格式化字符串写入固定大小的数组，处理截断。

------

关键组件

1. **头文件**

cpp

```cpp
#include <format>
#include <vector>
#include <string_view>
#include <iostream>
#include <iterator>
```

- <format>：提供 std::format_to 和 std::format_to_n。
- <vector>：提供 std::vector。
- <string_view>：提供 std::string_view。
- <iostream>：提供 std::cout。
- <iterator>：提供 std::back_inserter 和 std::ostream_iterator。
- **std::format_to 到动态缓冲区**

cpp

```cpp
std::vector<char> buffer;
int x = 42;
std::format_to(std::back_inserter(buffer), "x == {}", x);
std::string_view str1(buffer.begin(), buffer.end());
std::cout << str1 << "\n";
```

- **buffer**：

  - 动态字符向量，初始为空。

- **std::format_to**：

  - 原型：format_to(out, format_str, args...)。
  - 将格式化结果写入输出迭代器 out。

- **调用**：

  - std::back_inserter(buffer)：追加迭代器，动态扩展 buffer。
  - 格式化 "x == {}"，用 x = 42 替换 {}。
  - 结果写入 buffer：{'x', ' ', '=', ' ', '4', '2'}。

- **str1**：

  - string_view 视图，表示 "x == 42"。

- **输出**：

  ```text
  x == 42
  ```

- **std::format_to 到输出流**

cpp

```cpp
std::format_to(std::ostream_iterator<char>(std::cout), "x == {}\n", x);
```

- **std::ostream_iterator**：

  - 将字符写入 std::cout。

- **调用**：

  - 格式化 "x == {}\n"，用 x = 42 替换 {}。
  - 直接输出到 std::cout。

- **输出**：

  ```text
  x == 42
  ```

- **std::format_to_n 到静态缓冲区**

cpp

```cpp
std::array<char, 32> static_buffer;
std::format_to_n(static_buffer.begin(), 32,
    "Today is {}, Expected temperature is {} Celsius",
    "Tuesday", 24);
std::string_view str2(static_buffer.begin(), static_buffer.end());
std::cout << str2 << "\n";
```

- **static_buffer**：

  - 固定大小数组，容量 32。

- **std::format_to_n**：

  - 原型：format_to_n(out, n, format_str, args...)。
  - 写入最多 n 个字符，超出部分截断。

- **调用**：

  - 格式化 "Today is {}, Expected temperature is {} Celsius"。
  - 参数："Tuesday" 和 24。
  - 完整字符串："Today is Tuesday, Expected temperature is 24 Celsius"（45 个字符）。
  - 限制为 32 字符，截断为 "Today is Tuesday, Expected tempe"。

- **str2**：

  - string_view 视图，表示截断结果。

- **输出**：

  ```text
  Today is Tuesday, Expected tempe
  ```

------

为什么这样工作？

1. **std::format_to**：
   - 使用迭代器接口，灵活支持动态缓冲区和流。
   - 无大小限制，依赖输出目标的容量。
2. **std::format_to_n**：
   - 指定最大字符数，防止缓冲区溢出。
   - 截断超出部分，保证安全性。
3. **string_view**：
   - 轻量视图，展示缓冲区内容。

------

输出

```text
x == 42
x == 42
Today is Tuesday, Expected tempe
```

------

使用场景

- **动态输出**：
  - 将格式化结果存储到可扩展容器。
- **流输出**：
  - 直接格式化到 cout 或文件流。
- **固定缓冲区**：
  - 在有限空间内安全格式化。

------

总结

- std::format_to 将 "x == 42" 写入 vector 和 cout。
- std::format_to_n 将长字符串截断为 32 字符，写入数组。
- 代码展示了格式化函数的灵活性和安全性。