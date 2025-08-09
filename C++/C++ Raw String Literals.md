# Raw String Literals

在 C++ 中，**原始字符串字面量（Raw String Literals）** 是 C++11 引入的一种字符串字面量表示方式，用于方便地表示包含特殊字符（如换行符、引号、反斜杠等）的字符串，而无需使用转义字符（如 \n、 \"）。它通过特殊的语法避免手动转义，使得代码更简洁、可读，特别适用于正则表达式、JSON、多行文本等场景。

以下是对 C++ 原始字符串字面量的详细解释：

------

定义

原始字符串字面量的基本语法是：

cpp

```cpp
R"delimiter(raw_content)delimiter"
```

- **R**：表示原始字符串的开始。
- **delimiter**：用户定义的分隔符（可选），用于避免内容与结束标记冲突。
- **raw_content**：字符串的原始内容，不解析转义字符。

基本形式

cpp

```cpp
R"(content)"
```

- 无自定义分隔符时，直接用括号 ()。

带分隔符形式

cpp

```cpp
R"delim(content)delim"
```

- delim 是自定义分隔符，最多 16 个字符，不能包含括号、反斜杠或空格。

------

行为

- **无转义**：
  - 原始字符串中的字符按字面量处理，不解析 \ 为转义序列。
  - 例如，\n 不会变成换行符，而是保持为两个字符 \ 和 n。
- **多行支持**：
  - 可以直接包含换行符，无需显式 \n。
- **分隔符**：
  - 如果内容中包含 )，可以通过自定义分隔符避免冲突。

------

示例代码

示例 1：基本用法

cpp

```cpp
#include <iostream>
#include <string>

int main() {
    std::string s = R"(Hello\nWorld)";
    std::cout << s << std::endl;
    return 0;
}
```

输出

```text
Hello\nWorld
```

- **解释**：
  - \n 未被解析为换行符，而是作为普通字符输出。

示例 2：多行字符串

cpp

```cpp
#include <iostream>
#include <string>

int main() {
    std::string s = R"(Line 1
Line 2
Line 3)";
    std::cout << s << std::endl;
    return 0;
}
```

输出

```text
Line 1
Line 2
Line 3
```

- **解释**：
  - 换行符直接嵌入字符串，无需 \n。

示例 3：含特殊字符

cpp

```cpp
#include <iostream>
#include <string>

int main() {
    std::string s = R"("Quotes" and \backslashes\)";
    std::cout << s << std::endl;
    return 0;
}
```

输出

```text
"Quotes" and \backslashes\
```

- **解释**：
  - 引号和反斜杠无需转义。

示例 4：带自定义分隔符

cpp

```cpp
#include <iostream>
#include <string>

int main() {
    std::string s = R"xyz(This contains ) and more)xyz";
    std::cout << s << std::endl;
    return 0;
}
```

输出

```text
This contains ) and more
```

- **解释**：
  - 使用 xyz 作为分隔符，避免内容中的 ) 与结束标记冲突。

------

与传统字符串的对比

| 特性     | 传统字符串字面量  | 原始字符串字面量   |
| -------- | ----------------- | ------------------ |
| 语法     | "content"         | R"(content)"       |
| 转义字符 | 需要（如 \n, \"） | 不需要，直接字面量 |
| 多行     | 需要 \n 或续行符  | 直接换行           |
| 特殊字符 | 需转义（如 \\）   | 无需转义           |
| 可读性   | 复杂内容较差      | 高，适合复杂字符串 |

传统写法

cpp

```cpp
std::string s = "Line 1\nLine 2\n\"Quoted\"";
```

原始写法

cpp

```cpp
std::string s = R"(Line 1
Line 2
"Quoted")";
```

------

使用场景

1. **正则表达式**：

   - 避免大量反斜杠转义。

   cpp

   ```cpp
   std::string regex = R"(\d+\.\d+)"; // 传统写法： "\\d+\\.\\d+"
   ```

2. **多行文本**：

   - 嵌入代码、HTML、JSON 等。

   cpp

   ```cpp
   std::string json = R"({
       "name": "Alice",
       "age": 30
   })";
   ```

3. **调试输出**：

   - 表示路径或格式化字符串。

   cpp

   ```cpp
   std::string path = R"(C:\Program Files\app)";
   ```

------

注意事项

1. **C++11 要求**：

   - 需要 -std=c++11 或更高版本。

2. **分隔符限制**：

   - 最多 16 个字符。
   - 不能包含 (、)、\ 或空格。
   - 如果内容中包含 )delimiter"，必须调整分隔符。

3. **编码**：

   - 支持 u8（UTF-8）、u（UTF-16）、U（UTF-32）前缀：

     cpp

     ```cpp
     auto s = u8R"(UTF-8 string)";
     ```

4. **性能**：

   - 编译期解析，无运行时开销。

------

时间复杂度

- **构造**：O(1)，编译期完成。
- **存储**：取决于字符串长度，与普通字符串相同。

------

总结

C++ 原始字符串字面量通过 R"(...)" 语法提供了一种无需转义的字符串表示方式，极大简化了多行文本和特殊字符的处理。它提高了代码可读性，适用于正则表达式、多行数据等场景。自 C++11 起可用，是现代 C++ 编程的重要工具。如果你有具体问题或想探讨用法，请告诉我！