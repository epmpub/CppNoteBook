#  C++26 饱和运算函数：std::add_sat、std::div_sat 和 std::saturate_cast

```C++
#include <numeric>
#include <limits>
#include <print>

int main() {
    int x = std::numeric_limits<int>::max();
    // x + 1 is UB
    int y = std::add_sat(x,1); // OK
    // y == x

    std::println("x == {}, x + 1 == {}", x, y);

    int a = std::numeric_limits<int>::min();
    // a / -1 is UB
    int b = std::div_sat(a,-1); // OK
    // b == std::numeric_limits<int>::max()

    std::println("a == {}, a / -1 == {}", a, b);

    uint16_t c = std::saturate_cast<uint16_t>(-1);
    // c == 0

    std::println("c == {}", c);
}
```

这段代码展示了 C++26（或实验性实现）中 <numeric> 头文件引入的饱和运算函数：std::add_sat、std::div_sat 和 std::saturate_cast。这些函数旨在处理整数运算中的溢出问题，避免未定义行为（UB）。以下是逐步解释。

------

代码概览

- 使用 std::numeric_limits 获取整数类型的极限值。
- 展示传统运算的 UB 和饱和运算的安全替代。

------

关键组件

1. **头文件**

cpp

```cpp
#include <numeric>
#include <limits>
```

- <numeric>：提供 std::add_sat、std::div_sat 和 std::saturate_cast（C++26）。
- <limits>：提供 std::numeric_limits 获取类型范围。
- **std::add_sat 示例**

cpp

```cpp
int x = std::numeric_limits<int>::max();
int y = std::add_sat(x, 1);
```

- **std::numeric_limits<int>::max()**：
  - 返回 int 的最大值（通常 2³¹-1 = 2147483647）。
- **传统加法**：
  - x + 1 超出 int 范围，导致溢出（UB）。
- **std::add_sat**：
  - 原型：add_sat(a, b)。
  - 执行饱和加法：若结果超出范围，返回极限值。
  - x + 1 > max，饱和为 max。
- **结果**：
  - y == 2147483647（即 x）。
- **std::div_sat 示例**

cpp

```cpp
int a = std::numeric_limits<int>::min();
int b = std::div_sat(a, -1);
```

- **std::numeric_limits<int>::min()**：
  - 返回 int 的最小值（通常 -2³¹ = -2147483648）。
- **传统除法**：
  - a / -1 = 2147483648，超出 int 最大值（UB）。
- **std::div_sat**：
  - 原型：div_sat(a, b)。
  - 执行饱和除法：若结果超出范围，返回极限值。
  - -2147483648 / -1 > max，饱和为 max。
- **结果**：
  - b == 2147483647（std::numeric_limits<int>::max()）。
- **std::saturate_cast 示例**

cpp

```cpp
uint16_t c = std::saturate_cast<uint16_t>(-1);
```

- **std::saturate_cast**：
  - 原型：saturate_cast<T>(value)。
  - 将值转换为目标类型 T，若超出范围，饱和到边界值。
- **调用**：
  - -1 转为 uint16_t（范围 [0, 65535]）。
  - -1 < 0，饱和为最小值 0。
- **结果**：
  - c == 0。

------

为什么这样工作？

1. **饱和运算**：
   - 传统运算溢出导致 UB（未定义行为）。
   - 饱和运算将结果限制在类型范围内：
     - 加法：min <= result <= max。
     - 除法：同上。
     - 转换：目标类型的 min 或 max。
2. **实现**：
   - 检查溢出后返回边界值，而不是环绕或抛异常。
3. **安全性**：
   - 避免 UB，确保程序行为可预测。

------

输出

- 无显式输出，但结果为：
  - y == 2147483647。
  - b == 2147483647。
  - c == 0。

------

使用场景

- **数值计算**：
  - 处理可能溢出的运算（如图像处理、音频处理）。
- **类型转换**：
  - 安全地将值映射到有限范围。
- **鲁棒性**：
  - 避免 UB 导致的崩溃或错误。

------

总结

- std::add_sat：饱和加法，max + 1 仍为 max。
- std::div_sat：饱和除法，min / -1 饱和为 max。
- std::saturate_cast：饱和转换，-1 到 uint16_t 为 0。
- 代码展示了饱和运算如何安全处理溢出。