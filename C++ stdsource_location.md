# C++ std::source_location

在 C++ 中，std::source_location 是 C++20 引入的一个工具，用于在编译时捕获调用点的源代码信息，例如文件名、函数名、行号和列号。它定义在 <source_location> 头文件中，主要用于日志记录、调试或错误报告等场景，提供了一种标准化的方式来获取调用位置的元数据，而无需依赖宏（如 __FILE__ 和 __LINE__）。

以下是对 std::source_location 的详细解释：

------

定义和基本概念

- **头文件**：<source_location>
- **类**：std::source_location 是一个轻量级结构体，封装了源代码位置信息。
- **主要用途**：
  - 替代传统的宏（如 __FILE__, __LINE__, __FUNCTION__）。
  - 在函数调用时捕获调用点的静态信息。
- **编译时计算**：信息在编译时确定，不涉及运行时开销。

------

成员函数

std::source_location 提供了以下只读成员函数来访问源代码信息：

1. **file_name()**：
   - 返回调用点的文件名（const char* 类型）。
2. **function_name()**：
   - 返回调用点的函数名（const char* 类型）。
3. **line()**：
   - 返回调用点的行号（uint_least32_t 类型）。
4. **column()**：
   - 返回调用点的列号（uint_least32_t 类型，可能不被所有编译器支持）。

------

创建方式

- **静态工厂函数**：
  - 使用 std::source_location::current() 创建对象。
  - 通常作为默认参数传递给函数，以捕获调用点信息。
- **限制**：
  - 只能在编译时生成，不能动态构造。
  - 不支持拷贝构造或赋值（仅支持默认构造和移动）。

------

示例代码

1. 基本用法

cpp

```cpp
#include <iostream>
#include <source_location>

void log(const std::string& message, 
         std::source_location loc = std::source_location::current()) {
    std::cout << "File: " << loc.file_name() 
              << ", Function: " << loc.function_name() 
              << ", Line: " << loc.line() 
              << ", Column: " << loc.column() 
              << " - Message: " << message << "\n";
}

int main() {
    log("Hello, world!"); // 捕获调用点信息
    return 0;
}
```

- **输出示例**（具体值依赖编译环境）：

  ```text
  File: main.cpp, Function: main, Line: 12, Column: 5 - Message: Hello, world!
  ```

- **说明**：

  - std::source_location::current() 在 log 调用时捕获 main 中的位置。
  - 默认参数确保调用点是 log 被调用的地方，而不是 log 函数内部。

- 在类中使用

cpp

```cpp
#include <iostream>
#include <source_location>

class MyClass {
public:
    void debug(const std::string& msg, 
               std::source_location loc = std::source_location::current()) {
        std::cout << loc.file_name() << ":" << loc.line() 
                  << " in " << loc.function_name() << " - " << msg << "\n";
    }
};

int main() {
    MyClass obj;
    obj.debug("Testing"); // 捕获调用点
    return 0;
}
```

- **输出示例**：

  ```text
  main.cpp:14 in main - Testing
  ```

- **说明**：

  - loc 捕获的是 obj.debug("Testing") 的位置，而不是 debug 函数定义处。

- 与异常结合

cpp

```cpp
#include <iostream>
#include <source_location>
#include <stdexcept>

void risky_function(int x, 
                    std::source_location loc = std::source_location::current()) {
    if (x < 0) {
        throw std::runtime_error(
            std::string("Error at ") + loc.file_name() + ":" + 
            std::to_string(loc.line()) + " - Negative value");
    }
}

int main() {
    try {
        risky_function(-1);
    } catch (const std::exception& e) {
        std::cout << e.what() << "\n";
    }
    return 0;
}
```

- **输出示例**：

  ```text
  Error at main.cpp:15 - Negative value
  ```

- **说明**：

  - 异常消息包含调用点的源代码信息，便于调试。

------

与传统宏的比较

| 特性       | std::source_location  | 宏（如 __FILE__, __LINE__)  |
| ---------- | --------------------- | --------------------------- |
| 类型安全   | 是（强类型）          | 否（预处理器字符串或整数）  |
| 封装性     | 提供统一的接口        | 分散的宏                    |
| 函数名支持 | 是（function_name()） | 依赖 __FUNCTION__（非标准） |
| 列号支持   | 是（视编译器支持）    | 通常无                      |
| 代码可读性 | 更高（面向对象）      | 较低（宏替换）              |
| 编译时计算 | 是                    | 是                          |

- **优势**：
  - std::source_location 更现代化，类型安全，避免宏的预处理器魔法。
  - 提供统一的接口，易于扩展。
- **劣势**：
  - 需要 C++20 支持。
  - 列号支持可能不完整（依赖编译器）。

------

注意事项

- **编译器支持**：
  - 需要 C++20 兼容的编译器（如 GCC 10+, Clang 11+, MSVC 2019 16.10+）。
  - 某些编译器可能不支持 column() 或返回默认值（如 0）。
- **性能**：
  - 完全在编译时计算，无运行时开销。
- **局限性**：
  - 只能捕获静态位置信息，不能动态修改。
  - 不适用于需要运行时位置信息的场景。

------

总结

std::source_location 是 C++20 提供的一个优雅工具，用于在编译时捕获源代码位置信息。它替代了传统的宏方式，提供了类型安全和面向对象的接口，非常适合日志记录、调试和错误报告等场景。通过将其作为默认参数使用，可以方便地追踪调用点，是现代 C++ 编程中的一个实用特性。