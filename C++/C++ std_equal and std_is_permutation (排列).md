# std::equal and std::is_permutation (排列)

std::is_permutation 是 C++ 标准库中的一个算法，定义在 <algorithm> 头文件中。它用于判断一个序列是否是另一个序列的排列（permutation），也就是说，两个序列是否包含完全相同的元素，只是顺序可能不同。

函数签名

cpp

```cpp
template <class ForwardIt1, class ForwardIt2>
bool is_permutation(ForwardIt1 first1, ForwardIt1 last1, ForwardIt2 first2);

template <class ForwardIt1, class ForwardIt2, class BinaryPredicate>
bool is_permutation(ForwardIt1 first1, ForwardIt1 last1, ForwardIt2 first2, BinaryPredicate p);
```

参数

1. first1, last1: 定义第一个序列的迭代器范围 [first1, last1)。
2. first2: 第二个序列的起始迭代器，第二个序列的长度假定为 last1 - first1。
3. p（可选）: 一个二元谓词（binary predicate），用于自定义比较两个元素是否相等。如果不提供，默认使用 operator==。

返回值

- 如果第二个序列是第一个序列的一个排列，则返回 true。
- 否则返回 false。

工作原理

std::is_permutation 会检查两个序列是否包含相同的元素（数量和值相同），而不要求它们的顺序一致。它通常通过统计每个元素出现的次数来实现这一点。

示例代码

示例 1：基本用法

cpp

```cpp
#include <iostream>
#include <algorithm>
#include <vector>

int main() {
    std::vector<int> v1 = {1, 2, 3, 4};
    std::vector<int> v2 = {4, 3, 2, 1};

    bool result = std::is_permutation(v1.begin(), v1.end(), v2.begin());
    std::cout << (result ? "是排列" : "不是排列") << std::endl; // 输出：是排列

    std::vector<int> v3 = {1, 2, 3, 5};
    result = std::is_permutation(v1.begin(), v1.end(), v3.begin());
    std::cout << (result ? "是排列" : "不是排列") << std::endl; // 输出：不是排列

    return 0;
}
```

示例 2：使用自定义比较器

cpp

```cpp
#include <iostream>
#include <algorithm>
#include <vector>

int main() {
    std::vector<int> v1 = {1, 2, 3};
    std::vector<int> v2 = {2, 3, 1};

    // 使用 lambda 表达式作为自定义比较器
    auto comp = [](int a, int b) { return a == b; };
    bool result = std::is_permutation(v1.begin(), v1.end(), v2.begin(), comp);
    std::cout << (result ? "是排列" : "不是排列") << std::endl; // 输出：是排列

    return 0;
}
```

注意事项

1. **长度要求**: 如果两个序列的长度不同，std::is_permutation 会假设第二个序列的长度与第一个序列相同（即 last1 - first1）。如果第二个序列实际长度不足，可能会导致未定义行为。
2. **时间复杂度**: 通常为 O(n²)，但在某些实现中（例如使用哈希表优化），可能达到 O(n)。具体复杂度取决于标准库的实现。
3. **迭代器要求**: 输入的迭代器必须至少是前向迭代器（Forward Iterator）。

实际应用

- 判断两个字符串是否是anagram（字母异位词）。
- 检查两个集合是否包含相同元素（忽略顺序）。
- 在算法问题中验证排列关系。

希望这个解释清楚地解答了你的疑问！如果还有其他问题，欢迎继续提问。

## 2 介绍

The two algorithms for comparing the content of two containers for equality: std::equal and std::is_permutation.

Containers of the same type can be easily compared using comparison operators.

When we need to compare the content of containers of different types, we can use the *std::equal* and *std::is_permutation* algorithms.

```
#include <vector>
#include <set>
#include <algorithm>

std::vector<int> data1{2, 1, 3, 4, 5};
std::vector<int> data2{2, 4, 1, 3, 5};

// Linear comparison:
bool cmp1 = std::equal(data1.begin(), data1.end(), data2.begin());
// cmp1 == false
bool cmp2 = (data1 == data2);
// cmp2 == false (same as std::equal if types match)

// Elements match but are potentially out of order:
bool cmp3 = std::is_permutation(data1.begin(), data1.end(),
                                data2.begin());
// cmp3 == true


std::set<int> data3{1, 2, 3, 4, 5};

// Linear comparison:
bool cmp4 = std::ranges::equal(data1, data3);
// cmp4 == false

// Elements match but are potentially out of order:
bool cmp5 = std::ranges::is_permutation(data1, data3);
// cmp5 == true
```

[Open the example in Compiler Explorer.](https://compiler-explorer.com/z/oEzr5fhvE)