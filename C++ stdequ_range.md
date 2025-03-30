# C++ std::equ_range

std::equal_range 是 C++ 标准模板库 (STL) 中的一个算法函数，通常用于在已排序的容器中查找与某个特定值相等的所有元素的范围。它返回一对迭代器，表示这个范围的起始和结束位置。std::equal_range 特别适用于像 std::set 或 std::vector 这样支持随机访问或至少是已排序的序列。

定义

std::equal_range 位于 <algorithm> 头文件中。它的典型声明如下：



```cpp
template <class ForwardIterator, class T>
std::pair<ForwardIterator, ForwardIterator> equal_range(
    ForwardIterator first, 
    ForwardIterator last, 
    const T& value
);
```

或者带有自定义比较器的版本：



```cpp
template <class ForwardIterator, class T, class Compare>
std::pair<ForwardIterator, ForwardIterator> equal_range(
    ForwardIterator first, 
    ForwardIterator last, 
    const T& value, 
    Compare comp
);
```

参数

- first, last: 定义搜索范围的迭代器对，表示一个已排序的序列。
- value: 要查找的目标值。
- comp: 可选的比较函数，用于定义排序顺序（默认是 <）。

返回值

返回一个 std::pair，其中：

- first 是范围的**下界**（即不小于 value 的第一个元素的位置，等价于 std::lower_bound）。
- second 是范围的**上界**（即大于 value 的第一个元素的位置，等价于 std::upper_bound）。

如果没有找到相等的元素，则返回的范围是空的（即 first == second）。

前提条件

- 输入范围 [first, last) 必须是已排序的（默认按升序，或者按自定义比较器 comp 定义的顺序）。
- 如果未排序，结果是未定义的。

示例代码

以下是一个简单的例子，展示如何使用 std::equal_range：



```cpp
#include <iostream>
#include <vector>
#include <algorithm>

int main() {
    std::vector<int> vec = {1, 2, 2, 2, 3, 4, 5};

    // 查找所有值为 2 的元素
    auto range = std::equal_range(vec.begin(), vec.end(), 2);

    // 输出结果
    std::cout << "范围下界: " << *range.first << std::endl;  // 第一个 2
    std::cout << "范围上界: " << *(range.second - 1) << std::endl; // 最后一个 2

    // 输出所有匹配的元素
    std::cout << "匹配的元素: ";
    for (auto it = range.first; it != range.second; ++it) {
        std::cout << *it << " ";
    }
    std::cout << std::endl;

    return 0;
}
```

输出

```text
范围下界: 2
范围上界: 2
匹配的元素: 2 2 2
```

时间复杂度

- 对于随机访问迭代器（如 std::vector 或数组），时间复杂度是 **O(log n)**，因为它内部使用二分查找。
- 对于非随机访问迭代器（如 std::list），时间复杂度是 **O(n)**。

与 lower_bound 和 upper_bound 的关系

std::equal_range 实际上是 std::lower_bound 和 std::upper_bound 的组合：

- range.first 等价于 std::lower_bound(first, last, value)。
- range.second 等价于 std::upper_bound(first, last, value)。

用途

- 查找某个值的所有出现位置。
- 在关联容器（如 std::set 或 std::map）中统计某个键的出现次数。
- 在排序数组中进行范围查询。

希望这个解释清楚地回答了你的问题！如果有进一步的疑问，欢迎继续提问。

## 如果输入范围是未排序的,如何处理?

如果输入范围是未排序的，std::equal_range 的行为是未定义的，因为它依赖于二分查找，而二分查找要求序列已经按照某种顺序排序（默认是升序，或者按自定义比较器定义的顺序）。在未排序的情况下直接使用 std::equal_range，可能会返回错误的结果，甚至导致程序崩溃。因此，我们需要采取其他方法来处理未排序序列。

以下是处理未排序序列的几种方案，具体取决于你的需求：

------

1. ### 先排序再使用 std::equal_range

最直接的办法是将序列排序，然后再调用 std::equal_range。这种方法适用于你只需要查找一次，并且可以修改原始数据的情况。

示例代码



```cpp
#include <iostream>
#include <vector>
#include <algorithm>

int main() {
    std::vector<int> vec = {3, 1, 4, 1, 5, 2, 2};

    // 先排序
    std::sort(vec.begin(), vec.end());

    // 使用 equal_range 查找值为 2 的范围
    auto range = std::equal_range(vec.begin(), vec.end(), 2);

    // 输出结果
    std::cout << "匹配的元素: ";
    for (auto it = range.first; it != range.second; ++it) {
        std::cout << *it << " ";
    }
    std::cout << std::endl;

    return 0;
}
```

输出

```text
匹配的元素: 2 2
```

时间复杂度

- 排序：O(n log n)（std::sort 的复杂度）。
- std::equal_range：O(log n)。
- 总复杂度：O(n log n)。

注意

- 如果你不能修改原始数据，可以先复制一份副本进行排序。

------

2. ### 使用线性搜索（std::find 或循环）

如果不能排序，或者只需要查找一次，且数据量不大，可以使用线性搜索来找到所有匹配的元素。std::find 可以用来定位第一个匹配项，然后继续遍历找到所有匹配项。

示例代码



```cpp
#include <iostream>
#include <vector>
#include <algorithm>

int main() {
    std::vector<int> vec = {3, 1, 4, 1, 5, 2, 2};
    int target = 2;

    auto it = vec.begin();
    std::cout << "匹配的元素: ";
    while (it != vec.end()) {
        it = std::find(it, vec.end(), target);
        if (it != vec.end()) {
            std::cout << *it << " ";
            ++it;
        }
    }
    std::cout << std::endl;

    return 0;
}
```

输出

```text
匹配的元素: 2 2
```

时间复杂度

- 每次 std::find 是 O(n)，最坏情况下需要调用多次，总复杂度为 O(n * k)，其中 k 是匹配元素的数量。
- 通常可以优化为单次遍历，复杂度降为 O(n)。

优化版本（单次遍历）



```cpp
#include <iostream>
#include <vector>

int main() {
    std::vector<int> vec = {3, 1, 4, 1, 5, 2, 2};
    int target = 2;

    std::cout << "匹配的元素: ";
    for (int val : vec) {
        if (val == target) {
            std::cout << val << " ";
        }
    }
    std::cout << std::endl;

    return 0;
}
```

时间复杂度

- 单次遍历：O(n)。

------

3. 使用 std::count 检查出现次数

如果只需要知道某个值出现的次数，而不是具体位置，可以使用 std::count。

示例代码



```cpp
#include <iostream>
#include <vector>
#include <algorithm>

int main() {
    std::vector<int> vec = {3, 1, 4, 1, 5, 2, 2};
    int target = 2;

    int count = std::count(vec.begin(), vec.end(), target);
    std::cout << "值 " << target << " 出现了 " << count << " 次" << std::endl;

    return 0;
}
```

输出

```text
值 2 出现了 2 次
```

时间复杂度

- O(n)。

------

4. ### 使用哈希表（无序容器）

如果需要多次查询未排序序列中某个值的范围或出现次数，可以先将数据放入一个哈希表（如 std::unordered_multiset），然后利用哈希表的快速查找特性。

示例代码



```cpp
#include <iostream>
#include <vector>
#include <unordered_set>

int main() {
    std::vector<int> vec = {3, 1, 4, 1, 5, 2, 2};
    std::unordered_multiset<int> hash_set(vec.begin(), vec.end());

    int target = 2;
    auto range = hash_set.equal_range(target);

    std::cout << "匹配的元素: ";
    for (auto it = range.first; it != range.second; ++it) {
        std::cout << *it << " ";
    }
    std::cout << std::endl;

    return 0;
}
```

输出

```text
匹配的元素: 2 2
```

时间复杂度

- 构建哈希表：O(n)。
- 查询范围：平均 O(k)，其中 k 是匹配元素的数量。
- 总复杂度：O(n)（构建）+ O(k)（查询）。

注意

- std::unordered_multiset 的 equal_range 不依赖排序，但它的迭代器顺序是未定义的（不像排序容器那样连续）。

------

如何选择方法？

- **数据量小或只需一次查找**：用线性搜索（方案 2），简单直接。
- **可以修改数据且需要高效范围查询**：先排序再用 std::equal_range（方案 1）。
- **只需计数**：用 std::count（方案 3）。
- **需要多次查询**：用哈希表（方案 4）。

根据你的具体需求（例如是否可以排序、查询频率、数据规模），选择合适的方案。如果还有具体场景或问题，可以进一步讨论！