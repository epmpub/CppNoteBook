# std::nth_element

std::nth_element 是 C++ 标准模板库 (STL) 中的一个算法函数，定义在 <algorithm> 头文件中。它用于对范围内的元素进行部分排序，使得第 n 个元素（由迭代器指定）位于其最终排序位置，而小于它的元素在其左侧，大于它的元素在其右侧。std::nth_element 不保证整个范围完全排序，因此比完全排序（如 std::sort）更快。

以下是对 std::nth_element 的详细解释：

------

定义

cpp

```cpp
#include <algorithm>

template <class RandomAccessIterator>
void nth_element(RandomAccessIterator first, 
                 RandomAccessIterator nth, 
                 RandomAccessIterator last);

template <class RandomAccessIterator, class Compare>
void nth_element(RandomAccessIterator first, 
                 RandomAccessIterator nth, 
                 RandomAccessIterator last, 
                 Compare comp);
```

- **first, last**: 定义操作范围 [first, last) 的迭代器对。
- **nth**: 指向第 n 个元素的迭代器，表示要定位的元素。
- **comp**: 可选的比较函数，用于定义排序顺序（默认是 <）。
- **返回值**: 无（void）。

------

行为

- std::nth_element 重新排列范围 [first, last) 中的元素，使得：
  - *nth 是如果整个范围完全排序后位于该位置的元素。
  - [first, nth) 中的所有元素都小于或等于 *nth。
  - [nth + 1, last) 中的所有元素都大于或等于 *nth。
- 左侧和右侧的元素**不保证有序**，只是满足上述条件。

------

前提条件

- **迭代器类型**：
  - 必须是随机访问迭代器（RandomAccessIterator），如 std::vector 或数组的迭代器。
  - 不支持前向迭代器（如 std::list）。
- **范围有效性**：
  - [first, last) 必须有效，且 nth 在 [first, last) 内。

------

示例代码

示例 1：基本使用

cpp

```cpp
#include <iostream>
#include <vector>
#include <algorithm>

int main() {
    std::vector<int> vec = {5, 2, 9, 1, 5, 6};

    // 找到第 3 个元素（索引 2，从 0 开始）
    std::nth_element(vec.begin(), vec.begin() + 2, vec.end());

    std::cout << "第 3 个元素: " << vec[2] << "\n";
    std::cout << "调整后的向量: ";
    for (int x : vec) {
        std::cout << x << " ";
    }
    std::cout << std::endl;

    return 0;
}
```

输出（示例）

```text
第 3 个元素: 5
调整后的向量: 2 1 5 9 5 6
```

- 假设完全排序后是 {1, 2, 5, 5, 6, 9}，第 3 个元素是 5。
- 左侧 {2, 1} 小于等于 5，右侧 {9, 5, 6} 大于等于 5。

示例 2：自定义比较器

cpp

```cpp
#include <iostream>
#include <vector>
#include <algorithm>

int main() {
    std::vector<int> vec = {5, 2, 9, 1, 5, 6};

    // 按降序找到第 2 个元素（索引 1）
    std::nth_element(vec.begin(), vec.begin() + 1, vec.end(), std::greater<int>());

    std::cout << "第 2 个元素（降序）: " << vec[1] << "\n";
    std::cout << "调整后的向量: ";
    for (int x : vec) {
        std::cout << x << " ";
    }
    std::cout << std::endl;

    return 0;
}
```

输出（示例）

```text
第 2 个元素（降序）: 6
调整后的向量: 9 6 5 1 5 2
```

- 降序排序后是 {9, 6, 5, 5, 2, 1}，第 2 个元素是 6。

------

时间复杂度

- **平均情况**: O(n)，其中 n = last - first。
  - 通常使用快速选择算法（QuickSelect）的变种实现。
- **最坏情况**: O(n^2)，但在实际实现中很少发生（依赖随机化分区）。
- 比 std::sort（O(n log n)）更快，因为它只部分排序。

------

与其他算法的对比

| 特性       | std::nth_element  | std::sort  | std::partial_sort         |
| ---------- | ----------------- | ---------- | ------------------------- |
| 排序范围   | 仅定位第 n 个元素 | 整个范围   | 前 n 个元素               |
| 时间复杂度 | O(n)              | O(n log n) | O(n log k)（k 是前 n 个） |
| 结果       | *nth 在正确位置   | 完全排序   | 前 n 个排序               |
| 用途       | 找第 n 大/小      | 完全排序   | 前 k 大/小                |

------

使用场景

1. **查找第 k 大/小元素**：

   - 快速找到中位数、第 k 大值等。

   cpp

   ```cpp
   std::nth_element(vec.begin(), vec.begin() + vec.size()/2, vec.end()); // 中位数
   ```

2. **分区数据**：

   - 将数据分为小于和大于某值的两部分。

   cpp

   ```cpp
   std::nth_element(vec.begin(), vec.begin() + 2, vec.end()); // 前 3 小
   ```

3. **高效统计**：

   - 在大数据中快速定位特定百分位数。

------

注意事项

1. **不完全排序**：
   - 除了 *nth，其他元素只满足分区条件，不保证顺序。
2. **迭代器要求**：
   - 需要随机访问迭代器，否则无法使用。
3. **稳定性**：
   - std::nth_element 不保证稳定性（相等元素的相对顺序可能改变）。
4. **异常**：
   - 如果比较器 comp 抛出异常，行为未定义。

------

实现原理（概念性）

- std::nth_element 通常基于快速选择算法（QuickSelect）：
  1. 选择一个枢轴（pivot）。
  2. 将元素分区为小于、等于、大于枢轴的三部分。
  3. 根据 nth 的位置递归处理左侧或右侧。
- 与快速排序不同，它只递归处理一侧，因此平均复杂度为 O(n)。

------

总结

std::nth_element 是一个高效的算法，用于在 O(n) 时间内定位第 n 个元素并分区范围。它不完全排序，适合需要快速查找特定位置元素或分区的场景。与 std::sort 和 std::partial_sort 相比，它在性能和功能上提供了独特的选择。如果你有具体问题或使用案例，欢迎进一步讨论！