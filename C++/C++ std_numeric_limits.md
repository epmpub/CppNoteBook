# std::numeric_limits 

```C++
#include <limits>
#include <string>
#include <cstdint>
#include <print>

int main() {
    static_assert(std::numeric_limits<int>::is_exact == true);
    static_assert(std::numeric_limits<double>::is_exact == false);

    int v = std::numeric_limits<int>::max();
    // v == 2147483647 (for 32bit int)
    std::println("v == {}", v);

    // Query whether a type has a numeric_limits specialization:
    static_assert(
        std::numeric_limits<std::string>::is_specialized == false);

    int digits = std::numeric_limits<int64_t>::digits10;
    // digits == 18
    // int64_t can represent all 18 digit decimal numbers
    std::println("digits == {}", digits);
}
```



这段代码展示了 C++ 中 <limits> 头文件提供的 std::numeric_limits 模板，用于查询数值类型的属性。代码通过示例展示了整数和浮点数的精确性、最大值、十进制位数等特性，并验证了某些类型的特化是否存在。以下是逐步解释。

------

代码概览

- 使用 static_assert 检查类型的属性（如精确性和特化）。
- 查询 int 的最大值和 int64_t 的十进制位数。
- 使用 std::println 输出结果（C++23 特性）。

------

关键组件

1. **头文件**

cpp

```cpp
#include <limits>
#include <string>
#include <cstdint>
```

- <limits>：提供 std::numeric_limits，查询数值类型属性。
- <string>：用于测试非数值类型的特化。
- <cstdint>：提供固定大小整数类型（如 int64_t）。
- **精确性检查**

cpp

```cpp
static_assert(std::numeric_limits<int>::is_exact == true);
static_assert(std::numeric_limits<double>::is_exact == false);
```

- **std::numeric_limits<T>::is_exact**：
  - 表示类型是否为精确表示（无小数部分）。
- **int**：
  - 整数类型，无小数，is_exact == true。
- **double**：
  - 浮点类型，有精度损失，is_exact == false。
- **验证**：
  - static_assert 编译期检查，确保符合预期。
- **最大值查询**

cpp

```cpp
int v = std::numeric_limits<int>::max();
// v == 2147483647 (for 32bit int)
std::println("v == {}", v);
```

- **std::numeric_limits<int>::max()**：
  - 返回 int 的最大值。
  - 对于 32 位 int，范围是 [-2,147,483,648, 2,147,483,647]，最大值为 2,147,483,647。
- **std::println**：
  - C++23 格式化输出，等价于 std::cout << "v == " << v << "\n"。
- **结果**：
  - 输出：v == 2147483647（假设 32 位 int）。
- **特化检查**

cpp

```cpp
static_assert(
    std::numeric_limits<std::string>::is_specialized == false);
```

- **std::numeric_limits<T>::is_specialized**：
  - 表示是否有针对类型 T 的 numeric_limits 特化。
- **std::string**：
  - 非数值类型，无特化，is_specialized == false。
- **验证**：
  - 编译通过，确认 std::string 无特化。
- **十进制位数查询**

cpp

```cpp
int digits = std::numeric_limits<int64_t>::digits10;
// digits == 18
// int64_t can represent all 18 digit decimal numbers
std::println("digits == {}", digits);
```

- **std::numeric_limits<int64_t>::digits10**：
  - 表示类型能精确表示的最大十进制位数。
  - 对于 int64_t（64 位有符号整数），范围是 [-2^63, 2^63-1]，即 [-9,223,372,036,854,775,808, 9,223,372,036,854,775,807]。
  - 可表示所有 18 位十进制数（10^18 ≈ 1e18 < 2^63 ≈ 9.22e18），但 19 位会溢出。
- **结果**：
  - digits = 18。
  - 输出：digits == 18。

------

为什么这样工作？

1. **std::numeric_limits**：
   - 为内置数值类型（如 int、double、int64_t）提供特化，查询属性。
   - 非数值类型（如 std::string）无特化。
2. **精确性**：
   - 整数是精确的，浮点数有近似误差。
3. **最大值和位数**：
   - 依赖底层类型大小（32 位或 64 位）。
   - digits10 计算确保十进制表示无溢出。

------

输出

```text
v == 2147483647
digits == 18
```

- 假设 int 是 32 位，int64_t 是 64 位。

------

使用场景

- **类型检查**：
  - 验证类型属性（如精确性）。
- **范围限制**：
  - 获取最大值或最小值，防止溢出。
- **精度计算**：
  - 确定能表示的十进制位数，用于格式化或计算。

------

总结

- 代码展示了 std::numeric_limits 的用法：
  - int 是精确的，最大值 2,147,483,647（32 位）。
  - double 不精确。
  - std::string 无特化。
  - int64_t 可表示 18 位十进制数。
- 通过 static_assert 和 std::println，验证和展示了这些属性。