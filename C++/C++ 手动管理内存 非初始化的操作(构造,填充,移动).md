# 手动管理内存 非初始化的操作(构造,填充,移动)

```c++
#include <vector>
#include <string>
#include <memory>
#include <iostream>
#include <iomanip>

int main() {
    std::vector<std::string> src{"Hello", "World!"};
    { 
    void* buffer = std::aligned_alloc(alignof(std::string), sizeof(std::string) * src.size());
    if (buffer == nullptr) std::abort();
    auto raw_it = static_cast<std::string*>(buffer);

    { // Copy construction
    auto end_it = std::uninitialized_copy(src.begin(), src.end(), raw_it);
    // subrange(raw_it,end_it) == {"Hello", "World!"}

    for (auto &v : std::ranges::subrange(raw_it, end_it))
        std::cout << std::quoted(v) << "\n";
    
    // Manual creation requires manual destruction
    std::destroy(raw_it, end_it);
    }

    { // Copy construction from a single value
    auto end_it = raw_it + src.size();
    std::uninitialized_fill(raw_it, end_it, std::string("Something"));
    // subrange(raw_it,end_it) == {"Something", "Something"}

    for (auto &v : std::ranges::subrange(raw_it, end_it))
        std::cout << std::quoted(v) << "\n";
    
    // Manual creation requires manual destruction
    std::destroy(raw_it, end_it);
    }
    { // (C++17) Move construction
    auto end_it = raw_it + src.size();
    std::uninitialized_move(src.begin(), src.end(), raw_it);
    // subrange(raw_it,end_it) == {"Hello", "World!"}
    // src == {"", ""}

    for (auto &v : std::ranges::subrange(raw_it, end_it))
        std::cout << std::quoted(v) << "\n";
    for (auto &v : src)
        std::cout << std::quoted(v) << "\n";
    
    // Manual creation requires manual destruction
    std::destroy(raw_it, end_it);
    }

    // Free the buffer
    std::free(buffer);
    }

    { // C++20
    constexpr size_t size = 7;
    void* buffer = std::aligned_alloc(alignof(int), sizeof(int) * size);
    if (buffer == nullptr) std::abort();
    auto raw_it = static_cast<int*>(buffer);
    auto end_it = raw_it + size;

    // Value construction (for POD types this means zero-initialization)
    std::uninitialized_value_construct(raw_it, end_it);
    // subrange(raw_it, end_it) == {0, 0, 0, 0, 0, 0, 0}

    for (auto &v : std::ranges::subrange(raw_it, end_it))
        std::cout << v << " ";
    std::cout << "\n";

    // For the next example
    *raw_it = 42;

    // Manual creation requires manual destruction
    std::destroy(raw_it, end_it);

    // Default construction (for POD types this means no initialization)
    std::uninitialized_default_construct(raw_it, end_it);
    // The content is indeterminate values, however, in practical terms:
    // subrange(raw_it, end_it) == {42, 0, 0, 0, 0, 0, 0}

    for (auto &v : std::ranges::subrange(raw_it, end_it))
        std::cout << v << " ";
    std::cout << "\n";

    // Manual creation requires manual destruction
    std::destroy(raw_it, end_it);

    // Free the buffer
    std::free(buffer);
    }

}
```

我来详细解释这段 C++ 代码，它展示了如何使用手动内存管理和 C++ 标准库中的未初始化内存操作函数。

代码分为两大部分：一是处理 std::string 的操作，二是处理基本类型 int 的操作（C++20 特性）。

以下是逐步解释：

------

第一部分：处理 std::string

cpp

```cpp
std::vector<std::string> src{"Hello", "World!"};
void* buffer = std::aligned_alloc(alignof(std::string), sizeof(std::string) * src.size());
if (buffer == nullptr) std::abort();
auto raw_it = static_cast<std::string*>(buffer);
```

- **初始化**：
  - 创建一个包含 "Hello" 和 "World!" 的 std::vector<std::string>。
  - 使用 std::aligned_alloc 分配一块对齐的内存，大小为 sizeof(std::string) * 2，用于存储两个 std::string 对象。
  - 检查分配是否成功，若失败则调用 std::abort() 终止程序。
  - 将分配的内存转换为 std::string* 类型指针 raw_it。
- 复制构造

cpp

```cpp
auto end_it = std::uninitialized_copy(src.begin(), src.end(), raw_it);
for (auto &v : std::ranges::subrange(raw_it, end_it))
    std::cout << std::quoted(v) << "\n";
std::destroy(raw_it, end_it);
```

- **std::uninitialized_copy**：
  - 将 src 中的元素复制构造到未初始化的内存区域 raw_it 中。
  - 返回指向构造结束位置的迭代器 end_it。
  - 结果：subrange(raw_it, end_it) 包含 {"Hello", "World!"}。
- **输出**：遍历并打印每个字符串，带引号（使用 std::quoted）。
- **销毁**：手动调用 std::destroy 销毁构造的对象，因为它们是在未初始化内存上手动创建的。
- 填充构造

cpp

```cpp
auto end_it = raw_it + src.size();
std::uninitialized_fill(raw_it, end_it, std::string("Something"));
for (auto &v : std::ranges::subrange(raw_it, end_it))
    std::cout << std::quoted(v) << "\n";
std::destroy(raw_it, end_it);
```

- **std::uninitialized_fill**：
  - 用单一值 std::string("Something") 填充未初始化的内存区域。
  - 结果：subrange(raw_it, end_it) 包含 {"Something", "Something"}。
- **输出**：打印两个 "Something"。
- **销毁**：再次手动销毁对象。
- 移动构造 (C++17)

cpp

```cpp
auto end_it = raw_it + src.size();
std::uninitialized_move(src.begin(), src.end(), raw_it);
for (auto &v : std::ranges::subrange(raw_it, end_it))
    std::cout << std::quoted(v) << "\n";
for (auto &v : src)
    std::cout << std::quoted(v) << "\n";
std::destroy(raw_it, end_it);
```

- **std::uninitialized_move**：
  - 将 src 中的元素移动构造到未初始化的内存区域。
  - 结果：subrange(raw_it, end_it) 包含 {"Hello", "World!"}，而 src 变为 {"", ""}（字符串被移动后变为空）。
- **输出**：
  - 第一个循环打印新内存中的 "Hello" 和 "World!"。
  - 第二个循环打印 src 中的 "" 和 ""。
- **销毁**：销毁新构造的对象。

清理

cpp

```cpp
std::free(buffer);
```

- 释放通过 std::aligned_alloc 分配的内存。

------

第二部分：处理 int (C++20 特性)

cpp

```cpp
constexpr size_t size = 7;
void* buffer = std::aligned_alloc(alignof(int), sizeof(int) * size);
if (buffer == nullptr) std::abort();
auto raw_it = static_cast<int*>(buffer);
auto end_it = raw_it + size;
```

- **初始化**：
  - 分配一块对齐的内存，大小为 7 个 int。
  - 转换为 int* 类型指针 raw_it，end_it 指向末尾。
- 值构造

cpp

```cpp
std::uninitialized_value_construct(raw_it, end_it);
for (auto &v : std::ranges::subrange(raw_it, end_it))
    std::cout << v << " ";
std::cout << "\n";
```

- **std::uninitialized_value_construct**：
  - 对未初始化的内存执行值构造。
  - 对于 POD 类型（如 int），这意味着零初始化。
  - 结果：subrange(raw_it, end_it) 包含 {0, 0, 0, 0, 0, 0, 0}。
- **输出**：打印 0 0 0 0 0 0 0。
- 修改并默认构造

cpp

```cpp
*raw_it = 42;
std::destroy(raw_it, end_it);
std::uninitialized_default_construct(raw_it, end_it);
for (auto &v : std::ranges::subrange(raw_it, end_it))
    std::cout << v << " ";
std::cout << "\n";
```

- **修改**：将第一个元素设为 42。
- **销毁**：销毁之前的对象（对于 int 是空操作，但保持一致性）。
- **std::uninitialized_default_construct**：
  - 对未初始化的内存执行默认构造。
  - 对于 POD 类型（如 int），这意味着不初始化，值是不确定的。
  - 但由于内存未被清零，实际上保留了之前的值。
  - 结果：在实践中可能是 {42, 0, 0, 0, 0, 0, 0}（依赖实现）。
- **输出**：打印类似 42 0 0 0 0 0 0。

清理

cpp

```cpp
std::destroy(raw_it, end_it);
std::free(buffer);
```

- 销毁对象并释放内存。

------

关键点

1. **手动内存管理**：

   - 使用 std::aligned_alloc 和 std::free 管理内存。
   - 未初始化内存上的对象需要手动构造和销毁。

2. **构造方式**：

   - uninitialized_copy：复制构造。
   - uninitialized_fill：用单一值填充。
   - uninitialized_move：移动构造。
   - uninitialized_value_construct：值构造（零初始化 POD）。
   - uninitialized_default_construct：默认构造（POD 不初始化）。

3. **C++20 特性**：

   - 使用 std::ranges::subrange 方便地表示范围。
   - 新的构造函数提供更多控制。

4. **输出**：

   ```text
   "Hello"
   "World!"
   "Something"
   "Something"
   "Hello"
   "World!"
   ""
   ""
   0 0 0 0 0 0 0
   42 0 0 0 0 0 0
   ```

这段代码展示了如何在低级别管理内存并使用标准库工具安全地构造和销毁对象，非常适合理解 C++ 的内存模型和现代特性。