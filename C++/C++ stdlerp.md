# std::lerp

在 C++ 中，std::lerp 是一个数学函数，定义在 <cmath> 头文件中，于 C++20 引入。

The *std::lerp* is a C++20 mathematical function that handles linear interpolation (and extrapolation) for floating-point types.

它用于计算两个值之间的**线性插值**（linear interpolation，简称 lerp）。线性插值是一种常见技术，用于在两个已知点之间平滑过渡，常用于动画、图形学、数值计算等领域。

以下是对 std::lerp 的详细解释：

------

定义

- **头文件**：<cmath>

- **函数签名**：

  cpp

  ```cpp
  constexpr float lerp(float a, float b, float t) noexcept;
  constexpr double lerp(double a, double b, double t) noexcept;
  constexpr long double lerp(long double a, long double b, long double t) noexcept;
  ```

  - a：起始值。
  - b：结束值。
  - t：插值参数，通常在 [0, 1] 范围内。
  - 返回值：插值结果。

- **行为**：

  - 计算公式：a + t * (b - a)。
  - 当 t = 0 时，返回 a。
  - 当 t = 1 时，返回 b。
  - 当 t 在 (0, 1) 之间时，返回 a 和 b 之间的线性插值。

- **特性**：

  - constexpr：可在编译时计算。
  - noexcept：不会抛出异常。
  - 支持 float、double 和 long double 类型。

------

数学原理

线性插值的公式是：

```text
result = a + t * (b - a)
```

等价于：

```text
result = (1 - t) * a + t * b
```

- t 表示从 a 到 b 的进度：
  - t = 0：完全在 a。
  - t = 0.5：a 和 b 的中点。
  - t = 1：完全在 b。

------

示例代码

1. 基本用法

cpp

```cpp
#include <iostream>
#include <cmath>

int main() {
    float a = 0.0f;
    float b = 10.0f;

    std::cout << "t = 0.0: " << std::lerp(a, b, 0.0f) << "\n";
    std::cout << "t = 0.5: " << std::lerp(a, b, 0.5f) << "\n";
    std::cout << "t = 1.0: " << std::lerp(a, b, 1.0f) << "\n";
    return 0;
}
```

- **输出**：

  ```text
  t = 0.0: 0
  t = 0.5: 5
  t = 1.0: 10
  ```

- **说明**：在 0 和 10 之间插值，t 从 0 到 1 线性变化。

- 超出范围的 t

cpp

```cpp
#include <iostream>
#include <cmath>

int main() {
    double a = 2.0;
    double b = 4.0;

    std::cout << "t = -1.0: " << std::lerp(a, b, -1.0) << "\n";
    std::cout << "t = 2.0: " << std::lerp(a, b, 2.0) << "\n";
    return 0;
}
```

- **输出**：

  ```text
  t = -1.0: 0
  t = 2.0: 6
  ```

- **说明**：

  - t = -1.0：外推到 a 之前，2 + (-1) * (4 - 2) = 0。
  - t = 2.0：外推到 b 之后，2 + 2 * (4 - 2) = 6。

- 编译时计算

cpp

```cpp
#include <iostream>
#include <cmath>

constexpr double interpolate() {
    return std::lerp(10.0, 20.0, 0.25);
}

int main() {
    constexpr double result = interpolate();
    std::cout << "Result: " << result << "\n";
    return 0;
}
```

- **输出**：Result: 12.5
- **说明**：10 + 0.25 * (20 - 10) = 12.5，在编译时计算。

------

特殊情况

- **浮点精度**：
  - 当 t 接近 0 或 1 时，结果可能因浮点误差而略有偏差。
  - C++20 标准要求实现尽量减少这种误差。
- **NaN 和无穷大**：
  - 如果 a、b 或 t 是 NaN，结果是 NaN。
  - 如果 a 或 b 是无穷大，结果可能是无穷大或 NaN（依赖 t）。

示例：NaN

cpp

```cpp
#include <iostream>
#include <cmath>

int main() {
    std::cout << std::lerp(1.0f, 2.0f, std::nanf("")) << "\n";
    return 0;
}
```

- **输出**：nan
- **说明**：t 为 NaN，结果为 NaN。

------

与手写实现的比较

手写线性插值：

cpp

```cpp
float my_lerp(float a, float b, float t) {
    return a + t * (b - a);
}
```

- **std::lerp 的优势**：
  - 标准库实现，跨平台一致性。
  - constexpr 支持编译时计算。
  - 处理浮点误差更优化（C++20 要求）。
- **手写的好处**：
  - 更灵活，可自定义行为。

------

注意事项

- **C++20 支持**：需要编译器支持 C++20（如 GCC 9+、Clang 10+、MSVC 2019 16.6+）。
- **范围**：t 不限于 [0, 1]，可用于外推。
- **性能**：简单乘加运算，通常被内联，无显著开销。

------

总结

std::lerp 是 C++20 提供的一个简单而高效的线性插值函数，适用于数值过渡计算。它支持编译时计算，处理浮点精度问题，并与标准库风格一致。无论是动画插值、数据平滑还是数学计算，std::lerp 都是一个方便的工具，尤其在需要跨平台一致性时非常有用。