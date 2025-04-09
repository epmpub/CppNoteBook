# C++ std::istream std::ostream std::stringstream

```C++
#include <sstream>
#include <string>
#include <iomanip>
#include <iostream>
#include <type_traits>

int main() {
    std::istringstream in("10.17 22.3 \"Hello World\" 1234");
    float f1{}, f2{};
    std::string s;
    int i{};

    in >> f1 >> f2 >> std::quoted(s) >> i;
    // f1 == 10.17, f2 == 22.3, s == "Hello World", i == 1234

    std::cout << "f1 == " << f1 << ", f2 == " << f2 << ", s == " << std::quoted(s) << ", i == " << i << '\n';

    std::ostringstream out;

    out << f1 << " " << f2 << " " << std::quoted(s) << " " << i;
    auto str1 = out.str();
    // decltype(str1) == std::string
    // str1 == "10.17 22.3 \"Hello World\" 1234"

    static_assert(std::is_same_v<decltype(str1), std::string>);
    std::cout << "str1 == " << std::quoted(str1) << '\n';
    
    auto str2 = out.view(); // C++20
    // decltype(str2) == std::string_view
    // str2 == "10.17 22.3 \"Hello World\" 1234"

    static_assert(std::is_same_v<decltype(str2), std::string_view>);
    std::cout << "str2 == " << std::quoted(str2) << '\n';
}
```

这段代码展示了 C++ 中使用 <sstream>（字符串流）进行输入输出操作的典型用法，包括从字符串读取数据（std::istringstream）、将数据写入字符串（std::ostringstream），以及 C++14 的 std::quoted 和 C++20 的 std::string_view 特性。以下是对代码的详细解释：

------

**代码结构**

1. **头文件**：
   - <sstream>：提供 std::istringstream 和 std::ostringstream。
   - <string>：提供 std::string。
   - <iomanip>：提供 std::quoted。
   - <iostream>：提供 std::cout。
   - <type_traits>：提供 std::is_same_v 用于类型检查。
2. **主要内容**：
   - 从字符串流读取数据。
   - 将数据写入字符串流并提取结果。
   - 使用 std::quoted 处理带引号的字符串。
   - 验证类型并输出。

------

**代码逐步解释**

**1. 从字符串流读取数据**

cpp

```cpp
std::istringstream in("10.17 22.3 \"Hello World\" 1234");
float f1{}, f2{};
std::string s;
int i{};

in >> f1 >> f2 >> std::quoted(s) >> i;
// f1 == 10.17, f2 == 22.3, s == "Hello World", i == 1234
```

- **std::istringstream in**：
  - 创建输入字符串流，初始内容为 "10.17 22.3 \"Hello World\" 1234"。
  - 以空格分隔的序列，包含浮点数、带引号的字符串和整数。
- **变量初始化**：
  - f1 和 f2 为 float，默认初始化为 0.0。
  - s 为 std::string，默认空。
  - i 为 int，默认 0。
- **in >>**：
  - 流提取运算符，按顺序读取：
    - f1 = 10.17（浮点数）。
    - f2 = 22.3（浮点数）。
    - std::quoted(s)：读取带引号的字符串 "Hello World"，去除引号，存入 s。
    - i = 1234（整数）。
- **std::quoted**：
  - C++14 引入的操纵符，处理带引号的字符串输入输出。
  - 输入时：解析 "..."，去除引号，结果为 Hello World。

------

**2. 输出读取结果**

cpp

```cpp
std::cout << "f1 == " << f1 << ", f2 == " << f2 << ", s == " << std::quoted(s) << ", i == " << i << '\n';
```

- **输出**：

  - 将 f1、f2、s（带引号）和 i 输出。
  - std::quoted(s)：将 s 包裹在引号中，输出 "Hello World"。

- **结果**：

  ```text
  f1 == 10.17, f2 == 22.3, s == "Hello World", i == 1234
  ```

------

**3. 写入字符串流**

cpp

```cpp
std::ostringstream out;
out << f1 << " " << f2 << " " << std::quoted(s) << " " << i;
auto str1 = out.str();
// str1 == "10.17 22.3 \"Hello World\" 1234"
```

- **std::ostringstream out**：
  - 创建输出字符串流，用于构建字符串。
- **out <<**：
  - 依次写入：
    - f1 = 10.17。
    - 空格 " "。
    - f2 = 22.3。
    - 空格 " "。
    - std::quoted(s)：将 s 格式化为 "Hello World"（带引号）。
    - 空格 " "。
    - i = 1234。
- **out.str()**：
  - 返回流的内容作为 std::string。
  - str1 = "10.17 22.3 \"Hello World\" 1234"。
- **类型验证**：
  - decltype(str1) == std::string。

------

**4. 类型检查 str1**

cpp

```cpp
static_assert(std::is_same_v<decltype(str1), std::string>);
std::cout << "str1 == " << std::quoted(str1) << '\n';
```

- **static_assert**：

  - 验证 str1 的类型是 std::string。
  - std::is_same_v 是 C++17 的类型比较工具。

- **输出**：

  - std::quoted(str1)：将 str1 包裹在引号中。

  - 结果：

    ```text
    str1 == "10.17 22.3 \"Hello World\" 1234"
    ```

------

**5. 使用 std::string_view（C++20）**

cpp

```cpp
auto str2 = out.view();
static_assert(std::is_same_v<decltype(str2), std::string_view>);
std::cout << "str2 == " << std::quoted(str2) << '\n';
```

- **out.view()**：

  - C++20 引入，返回流的非拥有视图（std::string_view）。
  - 与 out.str() 的区别：
    - str() 返回 std::string，涉及内存拷贝。
    - view() 返回 std::string_view，避免拷贝，提高效率。
  - str2 = "10.17 22.3 \"Hello World\" 1234"。

- **static_assert**：

  - 验证 str2 的类型是 std::string_view。

- **输出**：

  - 结果：

    ```text
    str2 == "10.17 22.3 \"Hello World\" 1234"
    ```

------

**关键技术点**

1. **<sstream>**：
   - std::istringstream：从字符串读取数据。
   - std::ostringstream：将数据写入字符串。
2. **std::quoted**：
   - 处理带引号的字符串，输入时去除引号，输出时添加引号。
3. **std::string_view**：
   - C++20 的 ostringstream::view() 返回轻量视图，避免拷贝。
4. **类型检查**：
   - 使用 static_assert 和 std::is_same_v 确保类型正确性。

------

**输出总结**

```text
f1 == 10.17, f2 == 22.3, s == "Hello World", i == 1234
str1 == "10.17 22.3 \"Hello World\" 1234"
str2 == "10.17 22.3 \"Hello World\" 1234"
```

------

**可能的改进或注意事项**

1. **错误处理**：
   - 未检查 in >> 是否成功，可添加 if (in.fail())。
2. **格式控制**：
   - 可使用 <iomanip> 的 std::fixed 或 std::setprecision 控制浮点数输出。
3. **性能**：
   - str2 使用 string_view，适合只读场景，但生命周期依赖 out。

------

**总结**

- **输入**：从字符串流解析浮点数、字符串和整数。
- **输出**：将数据格式化为字符串，支持引号处理。
- **现代特性**：结合 C++14 的 std::quoted 和 C++20 的 std::string_view。
- **用途**：字符串处理、日志记录等场景。

如果你有具体问题（例如如何处理读取失败），欢迎提问！