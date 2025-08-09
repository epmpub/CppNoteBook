# *std::bit_and*, *std::bit_or*, *std::bit_xor* and *std::bit_not*

The four function objects *std::bit_and*, *std::bit_or*, *std::bit_xor* and *std::bit_not* model the functionality of the corresponding bit operators &, |, ^, ~.

As with all other function objects from the <functional> header, the void specialization will deduce the type from the arguments.



```C++
#include <functional>
#include <print>
#include <type_traits>

int main() {
    auto v1 = std::bit_and<unsigned>{}(255, 63);
    // decltype(v1) == unsigned, v1 == 63

    static_assert(std::is_same_v<decltype(v1), unsigned>);
    std::println("std::bit_and<unsigned>{{}}(255, 63) == {}", v1);

    auto v2 = std::bit_or<unsigned>{}(255, 63);
    // decltype(v2) == unsigned, v2 == 255

    static_assert(std::is_same_v<decltype(v2), unsigned>);
    std::println("std::bit_or<unsigned>{{}}(255, 63) == {}", v2);

    auto v3 = std::bit_xor<unsigned>{}(255, 63);
    // decltype(v3) == unsigned, v3 == 192

    static_assert(std::is_same_v<decltype(v3), unsigned>);
    std::println("std::bit_xor<unsigned>{{}}(255, 63) == {}", v3);

    auto v4 = std::bit_not<unsigned>{}(0);
    // decltype(v4) == unsigned, v4 == UINT_MAX

    static_assert(std::is_same_v<decltype(v4), unsigned>);
    std::println("std::bit_not<unsigned>{{}}(0) == {}", v4);

    // Deduced version
    auto v5 = std::bit_and<>{}(-1, -31);
    // decltype(v5) == int, v5 == -31

    static_assert(std::is_same_v<decltype(v5), int>);
    std::println("std::bit_and<>{{}}(-1, -31) == {}", v5);
}
```

这段代码展示了 C++ 中 <functional> 库提供的位操作函数对象（std::bit_and、std::bit_or、std::bit_xor 和 std::bit_not），它们封装了按位与、或、异或和取反操作。代码通过显式类型和类型推导两种方式使用这些函数对象，并验证其行为和返回类型。以下是对代码的详细解释：

------

**代码结构**

1. **头文件**：
   - <functional>：提供位操作函数对象。
   - <print>：C++23 的 std::println 用于格式化输出。
   - <type_traits>：提供 std::is_same_v 用于类型检查。
2. **主要内容**：
   - 使用显式类型（如 unsigned）调用位操作。
   - 使用类型推导调用位操作。
   - 验证结果和类型。

------

**代码逐步解释**

**1. std::bit_and<unsigned>（按位与）**

cpp

```cpp
auto v1 = std::bit_and<unsigned>{}(255, 63);
// v1 == 63
static_assert(std::is_same_v<decltype(v1), unsigned>);
std::println("std::bit_and<unsigned>{{}}(255, 63) == {}", v1);
```

- **std::bit_and<unsigned>**：

  - 模板参数指定操作类型为 unsigned。
  - 构造临时对象并调用其 operator()(T, T)。

- **计算**：

  - 255 = 0b11111111，63 = 0b00111111。
  - 255 & 63 = 0b00111111 = 63。

- **v1**：

  - 类型：unsigned，值：63。

- **输出**：

  ```text
  std::bit_and<unsigned>{}(255, 63) == 63
  ```

------

**2. std::bit_or<unsigned>（按位或）**

cpp

```cpp
auto v2 = std::bit_or<unsigned>{}(255, 63);
// v2 == 255
static_assert(std::is_same_v<decltype(v2), unsigned>);
std::println("std::bit_or<unsigned>{{}}(255, 63) == {}", v2);
```

- **std::bit_or<unsigned>**：

  - 按位或操作。

- **计算**：

  - 255 = 0b11111111，63 = 0b00111111。
  - 255 | 63 = 0b11111111 = 255。

- **v2**：

  - 类型：unsigned，值：255。

- **输出**：

  ```text
  std::bit_or<unsigned>{}(255, 63) == 255
  ```

------

**3. std::bit_xor<unsigned>（按位异或）**

cpp

```cpp
auto v3 = std::bit_xor<unsigned>{}(255, 63);
// v3 == 192
static_assert(std::is_same_v<decltype(v3), unsigned>);
std::println("std::bit_xor<unsigned>{{}}(255, 63) == {}", v3);
```

- **std::bit_xor<unsigned>**：

  - 按位异或操作。

- **计算**：

  - 255 = 0b11111111，63 = 0b00111111。
  - 255 ^ 63 = 0b11000000 = 192。

- **v3**：

  - 类型：unsigned，值：192。

- **输出**：

  ```text
  std::bit_xor<unsigned>{}(255, 63) == 192
  ```

------

**4. std::bit_not<unsigned>（按位取反）**

cpp

```cpp
auto v4 = std::bit_not<unsigned>{}(0);
// v4 == UINT_MAX
static_assert(std::is_same_v<decltype(v4), unsigned>);
std::println("std::bit_not<unsigned>{{}}(0) == {}", v4);
```

- **std::bit_not<unsigned>**：

  - 按位取反，单目运算符。

- **计算**：

  - 0 = 0b00000000（假设 32 位）。
  - ~0 = 0b11111111... = UINT_MAX（对于 unsigned，通常是 4294967295）。

- **v4**：

  - 类型：unsigned，值：UINT_MAX。

- **输出**：

  ```text
  std::bit_not<unsigned>{}(0) == 4294967295
  ```

------

**5. std::bit_and<>（类型推导）**

cpp

```cpp
auto v5 = std::bit_and<>{}(-1, -31);
// v5 == -31
static_assert(std::is_same_v<decltype(v5), int>);
std::println("std::bit_and<>{{}}(-1, -31) == {}", v5);
```

- **std::bit_and<>**：

  - 无显式类型，C++17 起支持从参数推导类型。
  - 参数 -1 和 -31 是 int，推导为 std::bit_and<int>。

- **计算**：

  - -1 = 0b11111111...（32 位补码）。
  - -31 = 0b11100001...。
  - -1 & -31 = 0b11100001... = -31。

- **v5**：

  - 类型：int，值：-31。

- **输出**：

  ```text
  std::bit_and<>{}(-1, -31) == -31
  ```

------

**关键技术点**

1. **函数对象**：
   - std::bit_and 等是函数对象，提供 operator() 执行位操作。
2. **模板参数**：
   - 可显式指定类型（如 unsigned），或推导（如 int）。
3. **位操作**：
   - 与（&）、或（|）、异或（^）、取反（~）。
4. **类型验证**：
   - static_assert 确保返回类型正确。
5. **C++23**：
   - std::println 简化格式化输出。

------

**输出总结**

```text
std::bit_and<unsigned>{}(255, 63) == 63
std::bit_or<unsigned>{}(255, 63) == 255
std::bit_xor<unsigned>{}(255, 63) == 192
std::bit_not<unsigned>{}(0) == 4294967295
std::bit_and<>{}(-1, -31) == -31
```

------

**可能的改进或注意事项**

1. **类型扩展**：
   - 可测试其他类型（如 uint64_t）。
2. **错误处理**：
   - 当前无异常，假设输入合法。
3. **进制输出**：
   - 可添加十六进制显示结果。

------

**总结**

- **功能**：展示 <functional> 的位操作函数对象。
- **用法**：支持显式类型和推导，适用于各种整数类型。
- **用途**：位运算、低级编程场景。

如果你有具体问题（例如其他位操作），欢迎提问！