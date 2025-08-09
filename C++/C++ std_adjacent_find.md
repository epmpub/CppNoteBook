# std::adjacent_find

std::adjacent_find 是 C++ 标准模板库 (STL) 中的一个算法函数，定义在 <algorithm> 头文件中。它用于在范围中查找**相邻的两个相同元素**（或满足特定条件的两个相邻元素），并返回指向第一个元素的迭代器。如果没有找到，则返回范围的结束迭代器。

以下是对 std::adjacent_find 的详细解释：

------

定义

cpp

```cpp
template <class ForwardIterator>
ForwardIterator adjacent_find(ForwardIterator first, ForwardIterator last);

template <class ForwardIterator, class BinaryPredicate>
ForwardIterator adjacent_find(ForwardIterator first, ForwardIterator last, BinaryPredicate pred);
```

- **first, last**: 定义搜索范围 [first, last) 的迭代器对。
- **pred**: 可选的二元谓词，用于自定义比较规则（默认是 ==）。
- **返回值**: 
  - 如果找到相邻的满足条件的元素，返回指向第一个元素的迭代器。
  - 如果没有找到，返回 last。

------

行为

- std::adjacent_find 检查范围 [first, last) 中每一对相邻元素（即 *it 和 *(it + 1)）。
- 默认情况下，它寻找第一个满足 *it == *(it + 1) 的位置。
- 如果提供了自定义谓词 pred，则寻找第一个满足 pred(*it, *(it + 1)) == true 的位置。

------

前提条件

- **迭代器类型**：
  - 必须是前向迭代器（ForwardIterator），如 std::vector、std::list 或数组的迭代器。
- **范围有效性**：
  - [first, last) 必须是有效的范围，且至少包含两个元素（否则无相邻元素可比较）。

------

示例代码

示例 1：查找相邻相等元素

cpp

```cpp
#include <iostream>
#include <vector>
#include <algorithm>

int main() {
    std::vector<int> vec = {1, 2, 2, 3, 4, 4, 5};

    auto it = std::adjacent_find(vec.begin(), vec.end());

    if (it != vec.end()) {
        std::cout << "找到相邻相等元素: " << *it << " 在位置 " 
                  << std::distance(vec.begin(), it) << std::endl;
    } else {
        std::cout << "未找到相邻相等元素" << std::endl;
    }

    return 0;
}
```

输出

```text
找到相邻相等元素: 2 在位置 1
```

示例 2：自定义谓词

cpp

```cpp
#include <iostream>
#include <vector>
#include <algorithm>

int main() {
    std::vector<int> vec = {1, 3, 2, 5, 4};

    // 查找相邻元素之差的绝对值小于等于 1
    auto it = std::adjacent_find(vec.begin(), vec.end(), 
                                 [](int a, int b) { return std::abs(a - b) <= 1; });

    if (it != vec.end()) {
        std::cout << "找到满足条件的相邻元素: " << *it << " 和 " << *(it + 1) 
                  << " 在位置 " << std::distance(vec.begin(), it) << std::endl;
    } else {
        std::cout << "未找到满足条件的相邻元素" << std::endl;
    }

    return 0;
}
```

输出

```text
找到满足条件的相邻元素: 3 和 2 在位置 1
```

------

时间复杂度

- **O(n)**：
  - 其中 n = last - first，算法最多检查 n - 1 对相邻元素。
  - 如果使用自定义谓词 pred，复杂度还取决于 pred 的执行时间。

------

使用场景

1. **查找重复元素**：
   - 检查序列中是否存在连续的相同值（如检测数据中的重复输入）。
2. **检测模式**：
   - 使用自定义谓词查找满足特定关系的相邻元素（如差值小于某个阈值）。
3. **数据分析**：
   - 在排序或未排序的序列中寻找局部特征。

------

注意事项

1. **范围长度**：
   - 如果范围为空或只有一个元素，std::adjacent_find 直接返回 last，因为没有相邻元素可比较。
2. **未定义行为**：
   - 如果 [first, last) 无效，或者迭代器不支持前向操作，可能导致未定义行为。
3. **默认比较**：
   - 默认使用 ==，确保元素类型支持此运算符，或者提供自定义谓词。

------

与其他算法的关系

- **std::find**：
  - 查找单个元素，而非相邻对。
- **std::unique**：
  - 删除相邻的重复元素，但需要先排序（std::adjacent_find 不要求排序）。

示例对比：

cpp

```cpp
std::vector<int> vec = {1, 2, 2, 3};
auto it1 = std::adjacent_find(vec.begin(), vec.end()); // 找到 2
auto it2 = std::find(vec.begin(), vec.end(), 2);       // 找到第一个 2
```

------

总结

std::adjacent_find 是一个简单高效的工具，用于在范围中查找相邻的相同元素或满足特定条件的元素对。它返回第一个匹配的位置，时间复杂度为线性，适用于各种序列分析任务。默认行为查找相等元素，但通过自定义谓词可以扩展到更复杂的逻辑。

如果你有具体问题或使用场景，欢迎进一步讨论！