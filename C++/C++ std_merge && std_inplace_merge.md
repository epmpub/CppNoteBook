# std::merge && std::inplace_merge



sample



```c++
#include <algorithm>
#include <functional>
#include <iostream>
#include <iterator>
#include <random>
#include <vector>
 
auto print = [](const auto rem, const auto& v)
{
    std::cout << rem;
    std::copy(v.begin(), v.end(), std::ostream_iterator<int>(std::cout, " "));
    std::cout << '\n';
};
 
int main()
{
    // fill the vectors with random numbers
    std::random_device rd;
    std::mt19937 mt(rd());
    std::uniform_int_distribution<> dis(0, 9);
 
    std::vector<int> v1(10), v2(10);
    std::generate(v1.begin(), v1.end(), std::bind(dis, std::ref(mt)));
    std::generate(v2.begin(), v2.end(), std::bind(dis, std::ref(mt)));
 
    print("Originally:\nv1: ", v1);
    print("v2: ", v2);
 
    std::sort(v1.begin(), v1.end());
    std::sort(v2.begin(), v2.end());
 
    print("After sorting:\nv1: ", v1);
    print("v2: ", v2);
 
    // merge
    std::vector<int> dst;
    std::merge(v1.begin(), v1.end(), v2.begin(), v2.end(), std::back_inserter(dst));
 
    print("After merging:\ndst: ", dst);
}
```



std::inplace_merge 是 C++ 标准模板库 (STL) 中的一个算法函数，定义在 <algorithm> 头文件中。它用于将两个已排序的子范围合并为一个单一的排序范围，且操作是在原地（in-place）完成的，不需要额外的存储空间（除了少量的临时变量）。这个函数特别适用于需要合并两个连续的已排序序列的场景。

以下是对 std::inplace_merge 的详细解释：

------

定义

cpp

```cpp
template <class BidirectionalIterator>
void inplace_merge(BidirectionalIterator first, 
                   BidirectionalIterator middle, 
                   BidirectionalIterator last);

template <class BidirectionalIterator, class Compare>
void inplace_merge(BidirectionalIterator first, 
                   BidirectionalIterator middle, 
                   BidirectionalIterator last, 
                   Compare comp);
```

- **first**: 整个范围的起始迭代器。
- **middle**: 分隔两个子范围的迭代器，第一个子范围是 [first, middle)，第二个子范围是 [middle, last)。
- **last**: 整个范围的结束迭代器。
- **comp**: 可选的比较函数，用于定义排序顺序（默认是 <）。
- **返回值**: 无（void）。

------

行为

- std::inplace_merge 假设：
  - [first, middle) 和 [middle, last) 是两个已排序的子范围（按相同的排序规则）。
- 它将这两个子范围合并为一个单一的排序范围 [first, last)。
- 合并过程是原地的，尽量减少额外内存使用（通常只使用少量临时空间）。

------

前提条件

1. **迭代器类型**：
   - 必须是双向迭代器（如 std::vector 或 std::list 的迭代器），因为算法需要前后移动。
2. **已排序**：
   - [first, middle) 和 [middle, last) 必须已经按相同的顺序（默认升序或由 comp 定义）排序。
   - 如果未排序，结果是未定义的。
3. **有效范围**：
   - [first, last) 必须是有效的范围，且 middle 在 [first, last) 内。

------

示例代码

示例 1：基本使用

cpp

```cpp
#include <iostream>
#include <vector>
#include <algorithm>

int main() {
    std::vector<int> vec = {1, 3, 5, 2, 4, 6};

    // 前半部分 [1, 3, 5] 和后半部分 [2, 4, 6] 已排序
    auto middle = vec.begin() + 3;
    std::inplace_merge(vec.begin(), middle, vec.end());

    // 输出合并后的结果
    for (int x : vec) {
        std::cout << x << " ";
    }
    std::cout << std::endl;

    return 0;
}
```

输出

```text
1 2 3 4 5 6
```

示例 2：自定义比较器

cpp

```cpp
#include <iostream>
#include <vector>
#include <algorithm>

int main() {
    std::vector<int> vec = {5, 3, 1, 6, 4, 2};

    // 前半部分 [5, 3, 1] 和后半部分 [6, 4, 2] 已按降序排序
    auto middle = vec.begin() + 3;
    std::inplace_merge(vec.begin(), middle, vec.end(), std::greater<int>());

    // 输出合并后的结果（降序）
    for (int x : vec) {
        std::cout << x << " ";
    }
    std::cout << std::endl;

    return 0;
}
```

输出

```text
6 5 4 3 2 1
```

------

时间复杂度

- **平均和最坏情况**: O(n log n)，其中 n = last - first。
  - 尽管是“原地”操作，std::inplace_merge 内部仍可能使用少量临时空间（通常是常数级别或对数级别），并通过复杂的移动和比较实现合并。
- **最佳情况**: 如果两个子范围已经部分重叠或接近合并状态，复杂度可能接近 O(n)。

相比之下，std::merge（需要额外空间）是 O(n)，但它不是原地的。

------

与 std::merge 的区别

| 特性       | std::merge             | std::inplace_merge     |
| ---------- | ---------------------- | ---------------------- |
| 空间需求   | 需要额外的输出范围     | 原地操作，少量临时空间 |
| 时间复杂度 | O(n)                   | O(n log n)             |
| 输入范围   | 可以是非连续的两个范围 | 必须是连续的单一范围   |
| 返回       | 输出到新范围，无返回值 | 修改原范围，无返回值   |

示例对比：

cpp

```cpp
std::vector<int> v1 = {1, 3, 5};
std::vector<int> v2 = {2, 4, 6};
std::vector<int> result(6);
std::merge(v1.begin(), v1.end(), v2.begin(), v2.end(), result.begin()); // 非原地
std::vector<int> v3 = {1, 3, 5, 2, 4, 6};
std::inplace_merge(v3.begin(), v3.begin() + 3, v3.end());              // 原地
```

------

使用场景

- **合并排序的中间步骤**：
  - std::inplace_merge 是合并排序（merge sort）算法的核心组件，用于合并两个已排序的子数组。
- **原地操作需求**：
  - 当内存受限，不希望分配额外空间时。
- **连续数据处理**：
  - 处理一个容器中已分段排序的部分。

------

注意事项

1. **未排序的输入**：
   - 如果 [first, middle) 或 [middle, last) 未排序，结果不可预测。
2. **性能权衡**：
   - 如果内存允许，std::merge 可能更快，因为它复杂度更低。
3. **迭代器要求**：
   - 需要双向迭代器，不支持仅前向迭代器（如 std::forward_list）。

------

总结

std::inplace_merge 是一个高效的原地合并算法，用于将两个已排序的连续子范围合并为一个排序范围。它在合并排序等场景中非常有用，但时间复杂度较高（O(n log n)），适用于内存受限或需要修改原数据的场合。如果你有具体问题或使用案例，欢迎进一步探讨！