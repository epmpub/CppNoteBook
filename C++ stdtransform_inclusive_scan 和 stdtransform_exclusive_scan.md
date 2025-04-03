# C++ std::transform_inclusive_scan 和 std::transform_exclusive_scan

在 C++ 中，std::transform_inclusive_scan 和 std::transform_exclusive_scan 是 C++17 引入的算法，定义在 <numeric> 头文件中。它们是并行友好的扫描（scan）操作，用于对输入范围的元素进行变换（transform）后计算前缀和（prefix sum），并将结果存储到输出范围。这两个函数在功能上类似，但扫描的方式不同（包含性 vs 排他性）。它们常用于数据处理、并行计算和累积操作。

以下是对这两个算法的详细解释：

------

共同特性

- **头文件**：<numeric>
- **并行支持**：可以通过执行策略（如 std::execution::par）启用并行执行（C++17）。
- **用途**：
  - 先对输入元素应用一元变换函数（unary operation）。
  - 然后对变换后的结果执行扫描操作（通常是累加）。
  - 将结果写入输出范围。
- **模板参数**：
  - 输入迭代器（输入范围）。
  - 输出迭代器（结果范围）。
  - 二元操作（binary operation，如加法）。
  - 一元操作（unary operation，变换函数）。
  - （可选）初始值（仅 transform_exclusive_scan 必需）。

------

std::transform_inclusive_scan

定义

- **函数签名**：

  cpp

  ```cpp
  template<class InputIt, class OutputIt, class BinaryOp, class UnaryOp>
  OutputIt transform_inclusive_scan(InputIt first, InputIt last, 
                                   OutputIt d_first, 
                                   BinaryOp binary_op, 
                                   UnaryOp unary_op);
  ```

  - first, last：输入范围。
  - d_first：输出范围的起始迭代器。
  - binary_op：二元操作（如加法）。
  - unary_op：一元变换函数。

- **行为**：

  - **包含性扫描（inclusive scan）**：每个输出元素是输入范围从头到当前位置（包括当前位置）的变换结果的累积。
  - 公式：对于输入 [x0, x1, x2, ...]，输出 [y0, y1, y2, ...]：
    - y[i] = binary_op(unary_op(x0), binary_op(unary_op(x1), ... unary_op(x[i])))。

示例

cpp

```cpp
#include <iostream>
#include <vector>
#include <numeric>

int main() {
    std::vector<int> input = {1, 2, 3, 4};
    std::vector<int> output(input.size());

    std::transform_inclusive_scan(
        input.begin(), input.end(), 
        output.begin(), 
        std::plus<>{},         // 二元操作：加法
        [](int x) { return x * 2; } // 一元操作：乘以 2
    );

    for (int x : output) {
        std::cout << x << " ";
    }
    std::cout << "\n";
    return 0;
}
```

- **计算过程**：
  - 输入：[1, 2, 3, 4]。
  - 变换后：[2, 4, 6, 8]（每个元素乘以 2）。
  - 包含性扫描：[2, 6, 12, 20]（2, 2+4, 2+4+6, 2+4+6+8）。
- **输出**：2 6 12 20

------

std::transform_exclusive_scan

定义

- **函数签名**：

  cpp

  ```cpp
  template<class InputIt, class OutputIt, class T, class BinaryOp, class UnaryOp>
  OutputIt transform_exclusive_scan(InputIt first, InputIt last, 
                                   OutputIt d_first, 
                                   T init, 
                                   BinaryOp binary_op, 
                                   UnaryOp unary_op);
  ```

  - 额外参数 init：初始值，用于计算第一个输出元素。

- **行为**：

  - **排他性扫描（exclusive scan）**：每个输出元素是输入范围从头到当前位置（不包括当前位置）的变换结果的累积，起始于初始值。
  - 公式：对于输入 [x0, x1, x2, ...]，输出 [y0, y1, y2, ...]：
    - y[0] = init。
    - y[i] = binary_op(init, binary_op(unary_op(x0), ... unary_op(x[i-1])))（i > 0）。

示例

cpp

```cpp
#include <iostream>
#include <vector>
#include <numeric>

int main() {
    std::vector<int> input = {1, 2, 3, 4};
    std::vector<int> output(input.size());

    std::transform_exclusive_scan(
        input.begin(), input.end(), 
        output.begin(), 
        0,                     // 初始值
        std::plus<>{},         // 二元操作：加法
        [](int x) { return x * 2; } // 一元操作：乘以 2
    );

    for (int x : output) {
        std::cout << x << " ";
    }
    std::cout << "\n";
    return 0;
}
```

- **计算过程**：
  - 输入：[1, 2, 3, 4]。
  - 变换后：[2, 4, 6, 8]。
  - 排他性扫描（初始值 0）：[0, 2, 6, 12]（0, 0+2, 0+2+4, 0+2+4+6）。
- **输出**：0 2 6 12

------

主要区别

| 特性       | transform_inclusive_scan  | transform_exclusive_scan           |
| ---------- | ------------------------- | ---------------------------------- |
| 扫描类型   | 包含性（包括当前位置）    | 排他性（不包括当前位置）           |
| 初始值     | 无需提供                  | 必须提供                           |
| 第一个元素 | 变换后的第一个输入值      | 初始值                             |
| 公式       | y[i] = sum(unary(x[0:i])) | y[i] = init + sum(unary(x[0:i-1])) |
| 典型用途   | 累积和包含当前元素        | 累积和仅基于前序元素               |

------

并行版本

两者都支持执行策略（C++17），例如：

cpp

```cpp
#include <execution>
// ...
std::transform_inclusive_scan(std::execution::par, 
                              input.begin(), input.end(), 
                              output.begin(), 
                              std::plus<>{}, 
                              [](int x) { return x * 2; });
```

- std::execution::par：并行执行。
- 要求 binary_op 是可交换和可结合的（如加法）。

------

注意事项

- **输入/输出范围**：
  - 输入和输出范围可以相同（原地操作），但需确保迭代器安全。
- **复杂度**：
  - 顺序执行：O(n)。
  - 并行执行：依赖实现，通常 O(n/p)（p 为线程数）。
- **异常**：
  - 如果 unary_op 或 binary_op 抛出异常，行为未定义。
- **要求**：
  - binary_op 必须是二元函数，且对变换后的值有效。
  - 输入范围必须有效，输出范围必须足够大。

------

总结

- **std::transform_inclusive_scan**：计算包含当前位置的变换前缀和，适合需要当前元素参与累积的场景。
- **std::transform_exclusive_scan**：计算不包含当前位置的变换前缀和，适合需要基于前序累积的场景。
- 两者结合了变换和扫描操作，支持并行执行，是现代 C++ 中处理数据序列的强大工具，尤其在高性能计算中非常有用。