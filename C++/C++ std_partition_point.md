# std::partition_point

在 C++ 中，std::partition_point 是标准库 <algorithm> 中提供的一个函数，用于在已分区（partitioned）的范围内查找分界点。它是二分查找的一种变体，专门用于处理满足某种条件的有序序列。以下是对 std::partition_point 的详细解释。

------

定义

std::partition_point 的典型声明如下：

cpp

```cpp
template<class ForwardIt, class UnaryPredicate>
ForwardIt partition_point(ForwardIt first, ForwardIt last, UnaryPredicate p);
```

- **参数**：
  - first, last：定义输入范围 [first, last) 的前向迭代器。
  - p：一元谓词（unary predicate），对范围中的元素返回 true 或 false。
- **返回**：一个迭代器，指向分区的分界点（即第一个不满足谓词 p 的元素）。
- **要求**：
  - 范围 [first, last) 必须是**已分区的**，也就是说，所有满足 p 的元素都位于不满足 p 的元素之前。
  - 迭代器必须是前向迭代器（ForwardIterator），支持二分查找。

------

工作原理

- std::partition_point 使用二分查找来高效定位分界点。
- **分区定义**：假设范围 [first, last) 被分为两部分：
  - [first, point)：所有元素满足 p(e) == true。
  - [point, last)：所有元素满足 p(e) == false。
  - point 是分界点，std::partition_point 返回这个位置。
- **复杂度**：对于大小为 N 的范围，时间复杂度是 O(log N)，因为它依赖二分查找。

------

示例代码

以下是一个使用 std::partition_point 的例子：

cpp

```cpp
#include <iostream>
#include <vector>
#include <algorithm>

int main() {
    std::vector<int> v = {1, 2, 3, 4, 5, 6, 7, 8, 9};
    
    // 使用 std::partition 将小于 5 的元素移到前面
    auto it = std::partition(v.begin(), v.end(), [](int x) { return x < 5; });
    
    // v 现在是分区后的结果，例如 {1, 2, 3, 4, 7, 6, 5, 8, 9}
    std::cout << "Partitioned vector: ";
    for (int x : v) std::cout << x << " ";
    std::cout << "\n";

    // 使用 std::partition_point 找到分界点
    auto point = std::partition_point(v.begin(), v.end(), [](int x) { return x < 5; });
    
    // 输出分界点位置和值
    std::cout << "Partition point at index: " << std::distance(v.begin(), point) << "\n";
    if (point != v.end()) {
        std::cout << "Value at partition point: " << *point << "\n";
    }

    return 0;
}
```

输出（可能的结果）

```text
Partitioned vector: 1 2 3 4 7 6 5 8 9
Partition point at index: 4
Value at partition point: 7
```

------

解释

1. **分区过程**：
   - std::partition 将 v 分区，使得所有小于 5 的元素（1, 2, 3, 4）位于前面，其他元素（5, 6, 7, 8, 9）位于后面。
   - 分区后的顺序可能是 {1, 2, 3, 4, 7, 6, 5, 8, 9}，具体顺序取决于实现，但保证前半部分满足条件。
2. **std::partition_point**：
   - 使用谓词 x < 5 查找分界点。
   - 返回的迭代器 point 指向第一个不满足 x < 5 的元素（这里是 7）。
   - 分界点索引为 4（从 0 开始计数）。
3. **结果**：
   - [v.begin(), point) 是满足 x < 5 的部分：{1, 2, 3, 4}。
   - [point, v.end()) 是不满足 x < 5 的部分：{7, 6, 5, 8, 9}。

------

与其他算法的关系

- **std::partition**：
  - 将范围分区，但不保证分界点位置，只返回一个迭代器。
  - 时间复杂度 O(N)。
- **std::partition_point**：
  - 假设范围已分区，高效查找分界点。
  - 时间复杂度 O(log N)。
- **类似算法**：
  - std::lower_bound：查找第一个不小于某值的元素。
  - std::upper_bound：查找第一个大于某值的元素。
  - std::partition_point 可以看作是对已分区范围的泛化版本。

------

使用场景

- **查找分界点**：在已分区的序列中快速定位满足条件的最后一个元素之后的位置。
- **二分查找扩展**：当数据已经被某种条件分割时，用于高效定位。
- **配合分区算法**：常与 std::partition 或 std::stable_partition 一起使用。

------

注意事项

1. **前提条件**：
   - 范围必须是已分区的，否则结果未定义。
   - 谓词 p 必须与分区的条件一致。
2. **迭代器要求**：
   - 需要前向迭代器，但通常在随机访问迭代器（如 std::vector）上效率最高。
3. **异常安全**：
   - 如果谓词不抛出异常，std::partition_point 提供强异常保证。

------

总结

std::partition_point 是一个高效的工具，用于在已分区范围内查找分界点。它利用二分查找的特性，适用于需要快速定位分区边界的场景。通过与 std::partition 配合使用，可以实现强大的数据处理逻辑。