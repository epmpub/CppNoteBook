# std::adjacent_difference

这段代码展示了 C++ 中 <numeric> 头文件中的 std::adjacent_difference 函数的多种用法。std::adjacent_difference 是一个用于计算相邻元素差值的算法，可以通过自定义操作实现不同的功能。以下是对代码的逐步解释。

```c++
#include <numeric>
#include <vector>
#include <execution>
#include <iostream>

int main() {
    std::vector<int> data{ 1, 2, 3, 4, 5, 6, 7 };

    // Strict left-fold operation: result = elem - old; old = move(elem);
    std::adjacent_difference(data.begin(), data.end(), data.begin());
    // data == {1, 1, 1, 1, 1, 1, 1}

    for (auto v : data)
        std::cout << v << " ";
    std::cout << "\n";

    // Generate Fibonacci sequence using a custom operation:
    std::adjacent_difference(data.begin(), std::prev(data.end()),
        std::next(data.begin()),
        [](int l, int r) { return l+r; });
    // data == {1, 1, 2, 3, 5, 8, 13}

    for (auto v : data)
        std::cout << v << " ";
    std::cout << "\n";

    // Non-linear version, output range cannot overlap input range.
    std::vector<double> average;
    std::adjacent_difference(std::execution::par_unseq,
        data.begin(), data.end(),
        std::back_inserter(average),
        [](int curr, int prev) {
            return (curr + prev) / 2.0;
        });
    // average == {1, 1, 1.5, 2.5, 4, 6.5, 10.5}

    for (auto v : average)
        std::cout << v << " ";
    std::cout << "\n";
}
```



------

**基础知识**

- **std::adjacent_difference**:
  - 计算输入范围内相邻元素之间的“差值”（或自定义操作的结果）。
  - 默认操作为减法：result[i] = data[i] - data[i-1]（对于 i > 0），而第一个元素保持不变。
  - 可以指定自定义二元操作代替减法。
- **参数**:
  - 输入迭代器范围：[first, last)。
  - 输出迭代器：结果写入的位置。
  - 可选：自定义操作（如 lambda 函数）。

------

**代码分解**

**1. 默认用法：严格左折叠**

cpp

```cpp
std::vector<int> data{ 1, 2, 3, 4, 5, 6, 7 };
std::adjacent_difference(data.begin(), data.end(), data.begin());
// data == {1, 1, 1, 1, 1, 1, 1}
```

- **操作**:
  - 默认使用减法：result[i] = data[i] - data[i-1]。
  - 第一个元素保持不变：result[0] = data[0]。
  - 输出覆盖输入范围（data.begin()）。
- **计算过程**:
  - 初始 data = {1, 2, 3, 4, 5, 6, 7}。
  - result[0] = data[0] = 1。
  - result[1] = data[1] - data[0] = 2 - 1 = 1。
  - result[2] = data[2] - data[1] = 3 - 2 = 1。
  - result[3] = data[3] - data[2] = 4 - 3 = 1。
  - ...
  - result[6] = data[6] - data[5] = 7 - 6 = 1。
- **结果**: data = {1, 1, 1, 1, 1, 1, 1}。
- **解释**: 每个元素变成了它与前一个元素的差，反映了“左折叠”行为。

------

**2. 生成斐波那契数列**

cpp

```cpp
std::adjacent_difference(data.begin(), std::prev(data.end()), 
    std::next(data.begin()),
    [](int l, int r) { return l + r; });
// data == {1, 1, 2, 3, 5, 8, 13}
```

- **操作**:
  - 输入范围：[data.begin(), std::prev(data.end()))，即 {1, 1, 1, 1, 1, 1}（前 6 个元素）。
  - 输出范围：从 std::next(data.begin()) 开始，即从第二个元素（索引 1）。
  - 自定义操作：l + r（加法）。
- **计算过程**:
  - 初始 data = {1, 1, 1, 1, 1, 1, 1}。
  - 输出从第二个元素开始：
    - data[1] = data[0] + data[1] = 1 + 1 = 1（保持不变）。
    - data[2] = data[1] + data[2] = 1 + 1 = 2。
    - data[3] = data[2] + data[3] = 2 + 1 = 3。
    - data[4] = data[3] + data[4] = 3 + 1 = 5。
    - data[5] = data[4] + data[5] = 5 + 1 = 8。
  - 最后一个元素 data[6] = 7 未被覆盖。
- **结果**: data = {1, 1, 2, 3, 5, 8, 13}。
- **解释**: 使用加法操作生成了斐波那契数列的前几项。每次计算时，l 是前一个结果，r 是原始值，逐步构建序列。

------

**3. 非线性版本：计算相邻平均值**

cpp

```cpp
std::vector<double> average;
std::adjacent_difference(std::execution::par_unseq,
    data.begin(), data.end(),
    std::back_inserter(average),
    [](int curr, int prev) {
        return (curr + prev) / 2.0;
    });
// average == {1, 1, 1.5, 2.5, 4, 6.5, 10.5}
```

- **操作**:
  - 输入范围：data = {1, 1, 2, 3, 5, 8, 13}。
  - 输出范围：average（通过 std::back_inserter 动态填充）。
  - 自定义操作：(curr + prev) / 2.0，计算相邻元素的平均值。
  - **std::execution::par_unseq**: 并行无序执行（C++17 特性），允许算法并行处理，但不保证顺序。
- **计算过程**:
  - average[0] = data[0] = 1（第一个元素直接复制）。
  - average[1] = (data[1] + data[0]) / 2.0 = (1 + 1) / 2.0 = 1。
  - average[2] = (data[2] + data[1]) / 2.0 = (2 + 1) / 2.0 = 1.5。
  - average[3] = (data[3] + data[2]) / 2.0 = (3 + 2) / 2.0 = 2.5。
  - average[4] = (data[4] + data[3]) / 2.0 = (5 + 3) / 2.0 = 4。
  - average[5] = (data[5] + data[4]) / 2.0 = (8 + 5) / 2.0 = 6.5。
  - average[6] = (data[6] + data[5]) / 2.0 = (13 + 8) / 2.0 = 10.5。
- **结果**: average = {1, 1, 1.5, 2.5, 4, 6.5, 10.5}。
- **解释**: 计算每对相邻元素的平均值，结果写入新向量。输入和输出范围不重叠，避免了覆盖问题。

------

**关键点**

1. **输入和输出范围**:
   - 如果输出范围与输入范围重叠（如第一个例子），结果会覆盖原始数据。
   - 如果不重叠（如第三个例子），需要确保输出容器有足够空间（如使用 std::back_inserter）。
2. **自定义操作**:
   - 默认是减法，但可以替换为任何二元操作（如加法、平均值）。
   - 参数顺序：对于 op(curr, prev)，curr 是当前元素，prev 是前一个元素。
3. **执行策略**:
   - std::execution::par_unseq 启用并行无序执行，适用于大数据集，但要求操作无副作用且线程安全。

------

**中文解释**

**1. 默认用法**

- 用减法计算相邻元素的差，结果覆盖原向量。
- 输入 {1, 2, 3, 4, 5, 6, 7} 变为 {1, 1, 1, 1, 1, 1, 1}。

**2. 斐波那契数列**

- 用加法代替减法，从第二个元素开始生成斐波那契数列。
- 输入 {1, 1, 1, 1, 1, 1, 1} 变为 {1, 1, 2, 3, 5, 8, 13}。

**3. 相邻平均值**

- 并行计算相邻元素的平均值，输出到新向量。
- 输入 {1, 1, 2, 3, 5, 8, 13} 产生 {1, 1, 1.5, 2.5, 4, 6.5, 10.5}。

------

**总结**

std::adjacent_difference 是一个灵活的工具：

- 默认行为是计算差值。
- 通过自定义操作，可以实现其他功能（如生成数列、计算平均值）。
- 支持并行执行，适用于性能敏感场景。

如果你有进一步问题或需要更多示例，请告诉我！