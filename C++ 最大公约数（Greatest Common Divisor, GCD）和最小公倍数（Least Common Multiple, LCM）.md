# 最大公约数（Greatest Common Divisor, GCD）和最小公倍数（Least Common Multiple, LCM）

这段代码展示了 C++17 中引入的 <numeric> 头文件中的两个函数：std::gcd 和 std::lcm，

分别用于计算两个整数的最大公约数（Greatest Common Divisor, GCD）和最小公倍数（Least Common Multiple, LCM）。

以下是逐步解释代码的工作原理。

------

完整代码

cpp

```cpp
#include <numeric>
#include <iostream>

int main() {   
    auto gcd = std::gcd(2*3, 3*5);
    // gcd == 3

    std::cout << "gcd(2*3, 3*5) == " << gcd << "\n";

    auto lcm = std::lcm(2*3, 3*5);
    // lcm == 2*3*5

    std::cout << "lcm(2*3, 3*5) == " << lcm << "\n";

    return 0;
}
```

------

逐步解释

**头文件 <numeric>**

- **std::gcd 和 std::lcm**：
  - C++17 引入，定义在 <numeric> 中。
  - 用于数值计算，提供数学上的 GCD 和 LCM。

------

**部分 1：std::gcd**

cpp

```cpp
auto gcd = std::gcd(2*3, 3*5);
// gcd == 3
```

- **std::gcd**：

  - 计算两个整数的最大公约数。

  - 签名：

    cpp

    ```cpp
    template<class M, class N>
    constexpr std::common_type_t<M, N> gcd(M m, N n);
    ```

  - 参数：

    - m = 2 * 3 = 6。
    - n = 3 * 5 = 15。

  - 返回类型：std::common_type_t<M, N>，这里是 int。

- **计算过程**：

  - 因子分解：
    - 6 = 2 * 3。
    - 15 = 3 * 5。
  - 公共因子：3。
  - GCD = 3。

- **结果**：

  - gcd = 3。

- **输出**：

  ```text
  gcd(2*3, 3*5) == 3
  ```

------

**部分 2：std::lcm**

cpp

```cpp
auto lcm = std::lcm(2*3, 3*5);
// lcm == 2*3*5
```

- **std::lcm**：

  - 计算两个整数的最小公倍数。

  - 签名：

    cpp

    ```cpp
    template<class M, class N>
    constexpr std::common_type_t<M, N> lcm(M m, N n);
    ```

  - 参数：

    - m = 2 * 3 = 6。
    - n = 3 * 5 = 15。

- **计算过程**：

  - 公式：LCM(a, b) = |a * b| / GCD(a, b)。
  - GCD(6, 15) = 3。
  - LCM(6, 15) = (6 * 15) / 3 = 90 / 3 = 30。
  - 或因子分解：
    - 6 = 2 * 3。
    - 15 = 3 * 5。
    - LCM = 2 * 3 * 5 = 30。

- **结果**：

  - lcm = 30（即 2 * 3 * 5）。

- **输出**：

  ```text
  lcm(2*3, 3*5) == 30
  ```

------

关键点分析

1. **std::gcd**：
   - 返回两个数的最大公约数。
   - 使用欧几里得算法（或类似优化）实现。
2. **std::lcm**：
   - 返回两个数的最小公倍数。
   - 基于 GCD 计算，避免直接分解因子。
3. **constexpr**：
   - 两者都支持编译期计算（若参数是常量表达式）。
4. **类型**：
   - 返回 std::common_type_t<M, N>，确保兼容输入类型。

------

时间复杂度

- **std::gcd**：
  - O(log(min(m, n)))，基于欧几里得算法。
- **std::lcm**：
  - 同上，加上乘法和除法，总体仍是 O(log(min(m, n)))。

------

使用场景

1. **数学计算**：
   - 简化分数、计算周期等。
2. **算法优化**：
   - 在需要公约数或公倍数的场景。
3. **编译期计算**：
   - 利用 constexpr 在编译时求值。

------

注意事项

1. **C++17 要求**：
   - 需要 -std=c++17 或更高版本。
2. **输入要求**：
   - 参数必须是整数类型（int, long, 等）。
   - 负数取绝对值，0 可能导致未定义行为（实现依赖）。
3. **溢出**：
   - std::lcm 可能溢出（如大数），需小心。

------

输出验证

运行代码：

```text
gcd(2*3, 3*5) == 3
lcm(2*3, 3*5) == 30
```

------

总结

- **std::gcd(6, 15)**：
  - 计算最大公约数，结果为 3。
- **std::lcm(6, 15)**：
  - 计算最小公倍数，结果为 30（2 * 3 * 5）。 C++17 的 <numeric> 提供了便捷的数学工具，直接支持 GCD 和 LCM 计算，简化了数值处理。如果你有具体问题或想扩展代码，请告诉我！