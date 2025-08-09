# *std::formatted_size* 带固定缓冲区

We have to first calculate the length of the formatted text using *std::formatted_size*.



```C++
#include <format>
#include <memory>
#include <numbers>
#include <iostream>

int main() {
    using namespace std::numbers;

    // calculate the required size to store the formatted text
    size_t sz = std::formatted_size("pi == {}", pi_v<double>);
    std::cout << sz << "\n";


    // allocate a big enough buffer (+ 1 for '\0')
    auto buffer = std::make_unique_for_overwrite<char[]>(sz + 1);

    // format text into the buffer
    std::format_to(buffer.get(), "pi == {}", pi_v<double>);
    buffer[sz] = '\0'; // terminate the string

    // buffer.get() == "pi == 3.141592653589793"

    std::cout << buffer.get() << "\n";
}
```

这段代码展示了 C++20 中引入的 <format> 库的使用，用于格式化字符串，并结合 <memory> 和 <numbers> 库来动态分配内存并访问数学常数。以下是对代码的详细解释：

------

**代码结构**

1. **头文件**：
   - <format>：提供 C++20 的格式化功能（如 std::format_to 和 std::formatted_size）。
   - <memory>：提供智能指针（如 std::make_unique_for_overwrite）。
   - <numbers>：C++20 中引入的数学常数库（如 std::numbers::pi_v）。
   - <iostream>：用于标准输出。
2. **主要功能**：
   - 计算格式化字符串的大小。
   - 动态分配缓冲区。
   - 格式化文本并输出。

------

**代码逐步解释**

cpp

```cpp
#include <format>
#include <memory>
#include <numbers>
#include <iostream>

int main() {
    using namespace std::numbers;

    // calculate the required size to store the formatted text
    size_t sz = std::formatted_size("pi == {}", pi_v<double>);

    // allocate a big enough buffer (+ 1 for '\0')
    auto buffer = std::make_unique_for_overwrite<char[]>(sz+1);
    
    // format text into the buffer
    std::format_to(buffer.get(), "pi == {}", pi_v<double>);
    buffer[sz] = '\0'; // terminate the string

    // buffer.get() == "pi == 3.141592653589793"

    std::cout << buffer.get() << "\n";
}
```

**1. using namespace std::numbers;**

- 将 <numbers> 命名空间引入作用域。
- <numbers> 提供数学常数，例如 pi_v<double> 表示 π 的双精度浮点值（约 3.141592653589793）。

**2. 计算格式化字符串大小**

cpp

```cpp
size_t sz = std::formatted_size("pi == {}", pi_v<double>);
```

- **std::formatted_size**：
  - 来自 <format> 库，计算格式化后的字符串长度（不包括末尾的 \0）。
  - 参数：
    - "pi == {}"：格式字符串，其中 {} 是占位符。
    - pi_v<double>：替换占位符的值，即 π 的双精度表示。
  - 返回值：格式化后的字符串 "pi == 3.141592653589793" 的长度。
  - 计算过程：
    - "pi == " 是 6 个字符。
    - 3.141592653589793 是 17 个字符（包括小数点）。
    - 总长度：6 + 17 = 23。
  - 因此，sz = 23。

**3. 动态分配缓冲区**

cpp

```cpp
auto buffer = std::make_unique_for_overwrite<char[]>(sz+1);
```

- **std::make_unique_for_overwrite**：
  - 来自 <memory>，C++20 引入的智能指针创建函数。
  - 与 std::make_unique 的区别：
    - std::make_unique 初始化内存（例如用 0 填充）。
    - std::make_unique_for_overwrite 不初始化内存，适用于将被覆盖的情况，效率更高。
  - 参数：sz+1，分配 24 个字符的数组（23 用于字符串，1 用于末尾的 \0）。
  - 返回值：std::unique_ptr<char[]>，管理动态分配的字符数组。
- **buffer**：
  - 智能指针，自动管理内存，避免手动 delete[]。

**4. 格式化到缓冲区**

cpp

```cpp
std::format_to(buffer.get(), "pi == {}", pi_v<double>);
buffer[sz] = '\0'; // terminate the string
```

- **std::format_to**：
  - 将格式化后的字符串写入指定位置。
  - 参数：
    - buffer.get()：目标缓冲区的起始地址（裸指针）。
    - "pi == {}"：格式字符串。
    - pi_v<double>：替换 {} 的值。
  - 功能：将 "pi == 3.141592653589793" 写入 buffer。
  - 注意：std::format_to 不自动添加 \0，只写入 sz 个字符。
- **buffer[sz] = '\0'**：
  - 手动添加字符串终止符，确保 buffer 是一个有效的 C 风格字符串。
  - sz = 23，所以 buffer[23] = '\0'。

**5. 输出结果**

cpp

```cpp
std::cout << buffer.get() << "\n";
```

- **buffer.get()**：

  - 返回缓冲区的裸指针，指向 "pi == 3.141592653589793"。

- **std::cout**：

  - 输出字符串并换行。

- **输出结果**：

  ```text
  pi == 3.141592653589793
  ```

**注释验证**

- 注释：buffer.get() == "pi == 3.141592653589793"。
  - 表示 buffer 中的内容与该字符串匹配，长度为 23（加上 \0 为 24）。

------

**关键技术点**

1. **<format> 库**：
   - C++20 引入的现代格式化工具，类似 Python 的 str.format()。
   - std::formatted_size 计算大小，std::format_to 写入缓冲区。
   - 比传统的 sprintf 或 std::stringstream 更高效且类型安全。
2. **std::make_unique_for_overwrite**：
   - 适用于将被覆盖的内存分配，避免不必要的初始化开销。
   - 智能指针确保内存安全释放。
3. **<numbers> 库**：
   - 提供编译期数学常数（如 pi_v<T>），类型安全且精确。
4. **手动终止字符串**：
   - std::format_to 不添加 \0，需要手动处理以兼容 C 风格字符串。

------

**运行流程总结**

1. 计算 "pi == 3.141592653589793" 的长度，得 sz = 23。
2. 分配 24 个字符的缓冲区（包含 \0）。
3. 将格式化字符串写入缓冲区，并添加 \0。
4. 输出结果："pi == 3.141592653589793"。

------

**可能的改进或注意事项**

- **直接使用 std::format**：
  - 如果不需要缓冲区，可以直接用 std::cout << std::format("pi == {}", pi_v<double>) << "\n";，更简洁。
- **缓冲区大小检查**：
  - 这里假设 std::formatted_size 准确，但在复杂格式中应验证输出是否超出预期。
- **C++23 <print>**：
  - 如果编译器支持 C++23，可以用 std::println("pi == {}", pi_v<double>); 替代 std::cout。

------

**总结**

这段代码展示了如何使用 C++20 的现代化工具（<format>、<memory>、<numbers>）来高效地格式化并输出数学常数 π。它强调了内存管理、格式化计算和字符串操作的结合。如果你有具体问题（例如某个函数的细节），欢迎进一步提问！