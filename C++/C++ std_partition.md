# std::partition

这段代码展示了 C++ 中两种分区（partition）算法的用法：传统的 std::partition 和 C++20 的 std::ranges::stable_partition。

它们都用于将范围中的元素按谓词（predicate）分为两部分，但实现方式和返回值有所不同。

我将逐部分解释代码的工作原理和差异。

------

完整代码（假设）

cpp

```cpp
#include <algorithm>
#include <vector>
#include <iostream>

int main() {
    // 使用 std::partition
    std::vector<int> data1{1, 2, 3, 4, 5, 6, 7, 8, 9};

    auto pp1 = std::partition(data1.begin(), data1.end(), 
        [](int v) { return v % 2 == 0; }); // 偶数谓词

    std::cout << "Even (partition): ";
    for (auto it = data1.begin(); it != pp1; ++it) {
        std::cout << *it << " ";
    }
    std::cout << "\nOdd (partition): ";
    for (auto it = pp1; it != data1.end(); ++it) {
        std::cout << *it << " ";
    }
    std::cout << "\n";

    // 使用 std::ranges::stable_partition
    std::vector<int> data2{1, 2, 3, 4, 5, 6, 7, 8, 9};

    auto odd = std::ranges::stable_partition(data2,
        [](int v) { return v % 2 == 0; }); // 偶数谓词

    auto even = std::ranges::subrange(data2.begin(), odd.begin());

    std::cout << "Even (stable): ";
    for (int x : even) {
        std::cout << x << " ";
    }
    std::cout << "\nOdd (stable): ";
    for (int x : odd) {
        std::cout << x << " ";
    }
    std::cout << "\n";

    return 0;
}
```

------

逐步解释

**部分 1：std::partition**

cpp

```cpp
std::vector<int> data1{1, 2, 3, 4, 5, 6, 7, 8, 9};

auto pp1 = std::partition(data1.begin(), data1.end(), 
    [](int v) { return v % 2 == 0; });
```

- **std::partition**：

  - 传统的分区算法，将范围 [begin, end) 分为两部分。
  - 谓词返回 true 的元素移到前面，false 的移到后面。
  - 不保证相对顺序（非稳定）。

- **谓词**：

  - [](int v) { return v % 2 == 0; }：
    - 返回 true 表示偶数，false 表示奇数。

- **执行过程**：

  - 初始：{1, 2, 3, 4, 5, 6, 7, 8, 9}。
  - 分区后：{2, 4, 6, 8, 1, 3, 5, 7, 9}（顺序可能不同）。
  - pp1 是分区点，指向第一个奇数（1）。

- **结果**：

  - [begin, pp1)：偶数，例如 {2, 4, 6, 8}。
  - [pp1, end)：奇数，例如 {1, 3, 5, 7, 9}。

- **遍历**：

  cpp

  ```cpp
  for (auto it = data1.begin(); it != pp1; ++it) {} // 偶数
  for (auto it = pp1; it != data1.end(); ++it) {}   // 奇数
  ```

输出（示例）

```text
Even (partition): 2 4 6 8
Odd (partition): 1 3 5 7 9
```

- **注意**：偶数和奇数的相对顺序可能改变。

------

**部分 2：std::ranges::stable_partition**

cpp

```cpp
std::vector<int> data2{1, 2, 3, 4, 5, 6, 7, 8, 9};

auto odd = std::ranges::stable_partition(data2,
    [](int v) { return v % 2 == 0; });
```

- **std::ranges::stable_partition**：
  - C++20 Ranges 版本的稳定分区算法。
  - 与 std::partition 类似，但保证元素相对顺序不变。
  - 返回一个 std::ranges::subrange，表示谓词为 false 的部分。
- **谓词**：
  - 同上，返回 true 表示偶数。
- **执行过程**：
  - 初始：{1, 2, 3, 4, 5, 6, 7, 8, 9}。
  - 分区后：{2, 4, 6, 8, 1, 3, 5, 7, 9}。
  - odd 是 {partition_point, end}，即 {1, 3, 5, 7, 9}。
- **结果**：
  - 偶数在前：{2, 4, 6, 8}。
  - 奇数在后：{1, 3, 5, 7, 9}。
  - 相对顺序保留。

**提取偶数**

cpp

```cpp
auto even = std::ranges::subrange(data2.begin(), odd.begin());
```

- **std::ranges::subrange**：
  - 表示一个子范围，[begin, partition_point)。
  - 这里是偶数部分：{2, 4, 6, 8}。

输出

```text
Even (stable): 2 4 6 8
Odd (stable): 1 3 5 7 9
```

- **注释**：
  - even == {2, 4, 6, 8}（顺序保证）。
  - odd == {1, 3, 5, 7, 9}（顺序保证）。

------

关键点分析

1. **std::partition**：
   - 非稳定分区，快速但不保留相对顺序。
   - 返回分区点迭代器。
2. **std::ranges::stable_partition**：
   - 稳定分区，保留相对顺序。
   - 返回奇数部分的 subrange，便于直接使用。
3. **谓词**：
   - 两者都用相同谓词，true（偶数）在前，false（奇数）在后。
4. **返回值**：
   - std::partition：单个迭代器。
   - std::ranges::stable_partition：subrange。

------

时间复杂度

- **std::partition**：
  - O(n)，n 是范围大小（9）。
- **std::ranges::stable_partition**：
  - O(n log n)（若内存允许），否则 O(n) 但需要额外空间。
  - 稳定性的代价。

------

使用场景

1. **快速分区**：
   - 用 std::partition 当顺序不重要。
2. **稳定分区**：
   - 用 std::ranges::stable_partition 当需要保持顺序。
3. **数据分组**：
   - 分离满足条件的元素。

------

注意事项

1. **C++20 要求**：
   - std::ranges::stable_partition 需要 -std=c++20。
2. **修改容器**：
   - 两者都直接修改 data1 和 data2。
3. **迭代器类型**：
   - 需要双向迭代器（vector 满足）。

------

总结

- **std::partition**：
  - 将 data1 分区为偶数在前，奇数在后，顺序不定。
  - 返回分区点，需手动拆分范围。
- **std::ranges::stable_partition**：
  - 将 data2 稳定分区，偶数在前，奇数在后，顺序保留。
  - 返回奇数 subrange，偶数通过 [begin, odd.begin()) 获取。 两种方法都实现了分区目标，但 Ranges 版本更现代，提供稳定性和便捷的范围处理。如果你有具体问题或想扩展代码，请告诉我！