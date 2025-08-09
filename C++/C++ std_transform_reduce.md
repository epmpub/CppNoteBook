# std::transform_reduce

std::transform_reduce 是 C++17 引入的一个算法函数，定义在 <numeric> 头文件中。它结合了变换（transform）和归约（reduce）的操作，用于对一个或两个范围的元素应用变换后进行归约（例如求和、乘积等）。它是现代 C++ 中并行化和泛型编程的重要工具，支持自定义操作和执行策略。

以下是对 std::transform_reduce 的详细解释：

------

定义

std::transform_reduce 有两种主要重载：

单范围版本

cpp

```cpp
#include <numeric>

template <class InputIt, class T, class BinaryOp, class UnaryOp>
T transform_reduce(InputIt first, InputIt last, T init, BinaryOp binary_op, UnaryOp unary_op);
```

- **first, last**: 输入范围 [first, last) 的迭代器对。
- **init**: 初始值，归约的结果从此开始。
- **binary_op**: 二元操作（如加法、乘法），用于归约。
- **unary_op**: 一元操作，变换每个元素。
- **返回值**: 变换并归约后的结果。

双范围版本

cpp

```cpp
template <class InputIt1, class InputIt2, class T, class BinaryOp1, class BinaryOp2>
T transform_reduce(InputIt1 first1, InputIt1 last1, InputIt2 first2, T init, 
                   BinaryOp1 binary_op, BinaryOp2 binary_op2);
```

- **first1, last1**: 第一个输入范围。
- **first2**: 第二个输入范围的起始迭代器（长度由第一个范围决定）。
- **init**: 初始值。
- **binary_op**: 外层归约操作。
- **binary_op2**: 内层变换操作，应用于每对元素。
- **返回值**: 变换并归约后的结果。

并行版本

cpp

```cpp
template <class ExecutionPolicy, class InputIt, class T, class BinaryOp, class UnaryOp>
T transform_reduce(ExecutionPolicy&& policy, InputIt first, InputIt last, T init, 
                   BinaryOp binary_op, UnaryOp unary_op);
```

- **policy**: 执行策略（如 std::execution::par），支持并行化。

------

行为

- **单范围**：
  - 对 [first, last) 中的每个元素应用 unary_op（变换）。
  - 将变换结果与 init 使用 binary_op 归约。
  - 等价于：init = binary_op(init, unary_op(*it)) 的累积。
- **双范围**：
  - 对 [first1, last1) 和 [first2, ...) 的每对元素应用 binary_op2（变换）。
  - 将变换结果与 init 使用 binary_op 归约。
  - 等价于：init = binary_op(init, binary_op2(*it1, *it2)) 的累积。

------

前提条件

- **迭代器类型**：
  - 必须是输入迭代器（InputIterator），如 std::vector 或数组的迭代器。
- **操作要求**：
  - binary_op 和 binary_op2 必须是可交换和可结合的（尤其在并行模式下）。
  - unary_op 必须返回可用于 binary_op 的类型。
- **范围有效性**：
  - [first, last) 或 [first1, last1) 必须有效，双范围时长度需匹配。

------

示例代码

示例 1：单范围（平方和）

cpp

```cpp
#include <iostream>
#include <vector>
#include <numeric>

int main() {
    std::vector<int> vec = {1, 2, 3, 4};

    // 计算平方和：1^2 + 2^2 + 3^2 + 4^2
    int result = std::transform_reduce(
        vec.begin(), vec.end(),
        0,                          // 初始值
        std::plus<>{},             // 加法归约
        [](int x) { return x * x; } // 平方变换
    );

    std::cout << "平方和: " << result << "\n";
    return 0;
}
```

输出

```text
平方和: 30
```

- 计算过程：0 + 1*1 + 2*2 + 3*3 + 4*4 = 0 + 1 + 4 + 9 + 16 = 30。

示例 2：双范围（点积）

cpp

```cpp
#include <iostream>
#include <vector>
#include <numeric>

int main() {
    std::vector<int> vec1 = {1, 2, 3};
    std::vector<int> vec2 = {4, 5, 6};

    // 计算点积：1*4 + 2*5 + 3*6
    int result = std::transform_reduce(
        vec1.begin(), vec1.end(), vec2.begin(),
        0,                          // 初始值
        std::plus<>{},             // 加法归约
        std::multiplies<>{}        // 乘法变换
    );

    std::cout << "点积: " << result << "\n";
    return 0;
}
```

输出

```text
点积: 32
```

- 计算过程：0 + 1*4 + 2*5 + 3*6 = 0 + 4 + 10 + 18 = 32。

示例 3：并行版本

cpp

```cpp
#include <iostream>
#include <vector>
#include <numeric>
#include <execution>

int main() {
    std::vector<int> vec = {1, 2, 3, 4, 5};

    // 并行计算平方和
    int result = std::transform_reduce(
        std::execution::par,       // 并行策略
        vec.begin(), vec.end(),
        0,
        std::plus<>{},
        [](int x) { return x * x; }
    );

    std::cout << "并行平方和: " << result << "\n";
    return 0;
}
```

输出

```text
并行平方和: 55
```

- 计算过程：0 + 1*1 + 2*2 + 3*3 + 4*4 + 5*5 = 0 + 1 + 4 + 9 + 16 + 25 = 55。

------

时间复杂度

- **顺序执行**：O(n)，其中 n 是范围长度。
- **并行执行**：O(n / p)，其中 p 是处理器核心数，实际取决于硬件和实现。
- 复杂度还受 binary_op 和 unary_op 的计算成本影响。

------

与其他算法的对比

| 特性     | std::transform_reduce | std::accumulate      | std::transform + std::reduce |
| -------- | --------------------- | -------------------- | ---------------------------- |
| 功能     | 变换 + 归约           | 仅归约               | 分步变换和归约               |
| 并行支持 | 是（C++17）           | 否（std::reduce 是） | 是（需手动组合）             |
| 灵活性   | 高（自定义操作）      | 中（仅归约）         | 高（分开执行）               |
| 效率     | 单步高效              | 简单场景更快         | 多步可能稍慢                 |

------

使用场景

1. **数学计算**：

   - 计算点积、平方和、范数等。

   cpp

   ```cpp
   double norm = std::transform_reduce(v.begin(), v.end(), 0.0, std::plus<>{}, 
                                       [](double x) { return x * x; });
   ```

2. **数据处理**：

   - 对数据应用变换后汇总。

   cpp

   ```cpp
   int sum_of_doubles = std::transform_reduce(v.begin(), v.end(), 0, std::plus<>{}, 
                                              [](int x) { return 2 * x; });
   ```

3. **并行优化**：

   - 在多核系统上加速大范围计算。

------

注意事项

1. **并行要求**：
   - binary_op 必须是可交换和可结合的（如加法、乘法），否则并行结果未定义。
2. **迭代器兼容性**：
   - 输入迭代器即可，但并行版本可能要求随机访问迭代器。
3. **C++17 要求**：
   - 需要支持 C++17 的编译器和标准库。
4. **异常**：
   - 如果操作抛出异常，结果未定义。

------

总结

std::transform_reduce 是 C++17 中一个高效的算法，结合变换和归约于一步，支持顺序和并行执行。它适用于数学计算、数据处理等场景，比单独使用 std::transform 和 std::reduce 更简洁。双范围版本尤其适合向量运算（如点积）。如果你有具体问题或想探讨某个用法，欢迎继续提问！