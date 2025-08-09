# std::getenv 获取环境变量



代码:

```C++
#include <cstdlib>
#include <ranges>
#include <print>

int main() {
    std::println("LD_LIBRARY_PATH:");

    // Get the LD_LIBRARY_PATH variable
    if (const char* record = std::getenv("LD_LIBRARY_PATH");
        record != nullptr) { // nullptr if the variable is not set

        std::string_view str = record;
        // Iterate over the chunks, delimited by ':'
        for (auto chunk : str | std::views::split(':')) {
            std::println("\t{}", std::string_view(chunk.begin(), chunk.end()));
        }
    }
}
```

这段代码展示了如何使用 C++ 的标准库功能来获取和解析环境变量 LD_LIBRARY_PATH，并利用 C++20 的 Ranges 库（<ranges>）分割字符串。代码通过示例展示了环境变量的处理和格式化输出。以下是逐步解释。

------

代码概览

- 使用 std::getenv 获取 LD_LIBRARY_PATH 环境变量。
- 使用 std::views::split 将字符串按冒号 : 分割。
- 使用 std::println 输出结果。

------

关键组件

1. **头文件**

cpp

```cpp
#include <cstdlib>
#include <ranges>
#include <print>
```

- <cstdlib>：提供 std::getenv。
- <ranges>：提供 std::views::split。
- <print>：C++23 的 std::println 用于格式化输出。
- **main 函数**

cpp

```cpp
int main() {
    std::println("LD_LIBRARY_PATH:");
    if (const char* record = std::getenv("LD_LIBRARY_PATH");
        record != nullptr) {
        std::string_view str = record;
        for (auto chunk : str | std::views::split(':')) {
            std::println("\t{}", std::string_view(chunk.begin(), chunk.end()));
        }
    }
}
```

**获取环境变量**

- **std::getenv**：
  - 原型：char* getenv(const char* name)。
  - 返回指向环境变量值的 C 字符串，若未设置则返回 nullptr。
- **if (const char\* record = std::getenv("LD_LIBRARY_PATH"); record != nullptr)**：
  - 获取 LD_LIBRARY_PATH。
  - 检查是否为 nullptr，避免未定义变量。

**字符串分割**

- **std::string_view str = record**：
  - 将 C 字符串转换为轻量视图。
- **str | std::views::split(':')**：
  - 使用 Ranges 库的 split 视图，按 : 分割 str。
  - 返回一个范围，每个元素是子范围（subrange），表示分隔符之间的部分。
- **for (auto chunk : ...)**：
  - 迭代每个子范围，chunk 是分割后的片段。

**输出**

- **std::string_view(chunk.begin(), chunk.end())**：
  - 将 chunk（subrange）转换为 string_view，便于输出。
- **std::println("\t{}", ...)**：
  - 格式化输出，每个片段前加制表符 \t。

------

执行流程

1. 输出标题：
   - LD_LIBRARY_PATH:。
2. 获取环境变量：
   - 若 LD_LIBRARY_PATH 未设置，程序结束。
   - 若设置（如 "path1:path2:path3"），继续处理。
3. 分割并输出：
   - 按 : 分割为 "path1"、"path2"、"path3"。
   - 逐行输出。

------

示例输出

假设 LD_LIBRARY_PATH = "/usr/lib:/usr/local/lib:/opt/lib"：

```text
LD_LIBRARY_PATH:
    /usr/lib
    /usr/local/lib
    /opt/lib
```

若未设置：

```text
LD_LIBRARY_PATH:
```

------

为什么这样工作？

1. **std::getenv**：
   - 提供对环境变量的低级访问，返回 C 字符串。
2. **std::string_view**：
   - 轻量包装，避免拷贝。
3. **std::views::split**：
   - 将字符串视图分割为子范围，现代且高效。
4. **std::println**：
   - 格式化输出，简化调试。

------

使用场景

- **环境变量解析**：
  - 处理路径列表（如 LD_LIBRARY_PATH、PATH）。
- **字符串处理**：
  - 使用 Ranges 分割字符串。
- **调试**：
  - 检查程序运行环境。

------

总结

- std::getenv 获取 LD_LIBRARY_PATH。
- std::views::split 按 : 分割为子范围。
- std::println 输出每个片段。
- 代码展示了现代 C++ 在环境变量处理中的应用。

其他:

![](D:\0.products\myBook\Programm_support_utilities.png)