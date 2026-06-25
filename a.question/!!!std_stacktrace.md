# std::stacktrace

```c++
#include <iostream>
#include <stacktrace>
#include <format>

auto fun = []() {
    // Obtain the current stacktrace.
    return std::stacktrace::current();
};

auto caller(auto&& c) {
    return c();
}
 
int main() {
    auto trace = caller(fun);
    for (const auto& entry : trace) {
        // Description, for GCC contains the name of the callable
        std::cout << "decription: " << 
            entry.description() << "\n";
        // Source file and line number
        std::cout << "file: " << entry.source_file() 
            << " at line: " << entry.source_line() << "\n\n";
    }
    std::cout << "\n";

    // Stacktrace supports stream insertion:
    std::cout << trace << "\n";
    // And formatted output:
    std::format_to(std::ostreambuf_iterator(std::cout),
        "{}\n", trace);
}
```



这段代码展示了 C++23 中引入的 <stacktrace> 头文件的功能，用于捕获和显示程序的调用栈信息。代码通过一个简单的例子演示了如何获取当前调用栈、遍历栈帧（stack frame）并输出详细信息。以下是对代码的逐步解释：

------

1. **代码结构**

cpp

```cpp
#include <iostream>
#include <stacktrace>
#include <format>
```

- **依赖**：
  - <iostream>：用于标准输出。
  - <stacktrace>：提供 std::stacktrace 类，捕获调用栈。
  - <format>：支持格式化输出（C++20 引入，C++23 完善）。

------

2. **Lambda 函数 fun**

cpp

```cpp
auto fun = []() {
    return std::stacktrace::current();
};
```

- **功能**：定义一个无参数的 lambda 函数，返回当前调用栈。
- **std::stacktrace::current()**：
  - 静态函数，捕获调用它时的栈跟踪信息。
  - 返回一个 std::stacktrace 对象，包含从当前点到程序入口的调用栈帧。
- **作用**：将栈跟踪捕获封装在 lambda 中，便于在其他地方调用。

------

3. **模板函数 caller**

cpp

```cpp
auto caller(auto&& c) {
    return c();
}
```

- **功能**：一个简单的模板函数，接受一个可调用对象（c），并调用它。
- **参数**：auto&& 使用通用引用（universal reference），支持左值和右值。
- **作用**：间接调用 fun，在调用栈中增加一层，展示多层调用的效果。

------

4. **main 函数**

cpp

```cpp
int main() {
    auto trace = caller(fun);
```

- **调用栈捕获**：
  - caller(fun) 调用 fun()，fun 在其内部捕获栈跟踪。
  - trace 是一个 std::stacktrace 对象，记录了从 fun 到程序入口的调用栈。

遍历栈帧

cpp

```cpp
for (const auto& entry : trace) {
    std::cout << "decription: " << entry.description() << "\n";
    std::cout << "file: " << entry.source_file() 
              << " at line: " << entry.source_line() << "\n\n";
}
```

- **trace**：std::stacktrace 是一个可迭代对象，每个元素是 std::stacktrace_entry。
- **std::stacktrace_entry 的成员函数**：
  - description()：返回栈帧的描述（通常是函数名，依赖实现）。
  - source_file()：返回源文件名。
  - source_line()：返回源代码行号。
- **输出**：逐帧打印调用栈的详细信息。

直接输出

cpp

```cpp
std::cout << trace << "\n";
```

- **std::stacktrace 支持流输出**：
  - 通过 operator<< 直接将整个栈跟踪输出到流。
  - 格式依赖于实现（如 GCC、MSVC）。

格式化输出

cpp

```cpp
std::format_to(std::ostreambuf_iterator(std::cout), "{}\n", trace);
```

- **使用 <format>**：
  - std::format_to 将 trace 格式化为字符串并输出。
  - {} 是占位符，trace 被格式化为实现定义的字符串。

------

5. **示例输出（GCC）**

```text
/* Example formatted output for GCC:
   0# fun::{lambda()#1}::operator()() const at /app/example.cpp:6
   1# auto caller<fun::{lambda()#1}&>(fun::{lambda()#1}&) at /app/example.cpp:10
   2# main at /app/example.cpp:14
   3#      at :0
   4# _start at :0
   5# 
*/
```

- **解释**：
  - **帧 0**：fun 的 lambda 的 operator()，在 example.cpp:6 调用 std::stacktrace::current()。
  - **帧 1**：caller 函数，模板实例化为 caller<lambda>，在 example.cpp:10 调用 c()。
  - **帧 2**：main 函数，在 example.cpp:14 调用 caller(fun)。
  - **帧 3 和 4**：运行时库的入口函数（如 _start），源信息不可用（:0）。
  - **帧 5**：空行，表示栈底。
- **实现依赖**：
  - GCC 使用符号名（如 fun::{lambda()#1}）和文件行号。
  - 其他编译器（如 MSVC）可能有不同格式。

------

6. **关键点**

1. **std::stacktrace::current()**：
   - 捕获当前调用栈，返回 std::stacktrace 对象。
   - 栈帧从当前函数向上追溯到程序入口。
2. **std::stacktrace_entry**：
   - 提供栈帧的详细信息（描述、文件、行号）。
   - 信息完整性依赖编译器和调试符号。
3. **输出方式**：
   - 遍历：细粒度控制每帧输出。
   - 流输出：简洁输出整个栈。
   - 格式化：结合 <format> 提供灵活性。
4. **用途**：
   - 调试：定位调用路径。
   - 日志：记录异常或关键点时的栈信息。

------

7. **注意事项**

- **编译器支持**：
  - <stacktrace> 是 C++23 特性，需要支持 C++23 的编译器（如 GCC 13+、MSVC 19.30+）。
  - 输出格式和细节因实现而异。
- **性能**：
  - 捕获栈跟踪可能有开销，适合调试或错误处理，不建议频繁使用。
- **调试信息**：
  - 需要编译时启用调试符号（如 -g），否则 source_file() 和 source_line() 可能为空。

------

总结

这段代码展示了如何使用 std::stacktrace 捕获和显示调用栈信息。通过 lambda、模板函数和多种输出方式，演示了其灵活性和实用性。示例输出反映了典型的调用栈结构，从用户代码到运行时入口，为调试和日志记录提供了强大支持。