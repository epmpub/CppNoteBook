#  std::set_union、std::set_intersection、std::set_difference 和 std::set_symmetric_difference

我来为你详细解释 C++ 标准库中的 std::set_union、std::set_intersection、std::set_difference 和 std::set_symmetric_difference。这些算法都定义在 <algorithm> 头文件中，用于对两个有序序列执行集合操作。它们通常用于处理已排序的容器（如 std::set 或手动排序的 std::vector）

------

前提条件

这些算法要求输入范围是**有序的**（默认按升序，使用 < 比较），否则结果未定义。如果需要自定义排序规则，可以提供比较器。

------

1. std::set_union

作用

计算两个有序序列的并集（合并所有元素，去除重复）。

语法

cpp

```cpp
template<class InputIterator1, class InputIterator2, class OutputIterator>
OutputIterator set_union(InputIterator1 first1, InputIterator1 last1,
                         InputIterator2 first2, InputIterator2 last2,
                         OutputIterator result);
```

- first1, last1: 第一个有序范围。
- first2, last2: 第二个有序范围。
- result: 输出迭代器，指向结果存储位置。
- 返回值: 指向结果范围末尾的迭代器。

示例

cpp

```cpp
#include <iostream>
#include <algorithm>
#include <vector>

int main() {
    std::vector<int> v1 = {1, 2, 3, 4};
    std::vector<int> v2 = {3, 4, 5, 6};
    std::vector<int> result;

    std::set_union(v1.begin(), v1.end(), v2.begin(), v2.end(),
                   std::back_inserter(result));

    for (int x : result)
        std::cout << x << " ";
    std::cout << "\n";
}
```

**输出:** 1 2 3 4 5 6

特点

- 结果包含两个范围中的所有元素，重复元素只出现一次。
- 时间复杂度: O(n + m)，其中 n 和 m 是两个输入范围的长度。

------

2. std::set_intersection

作用

计算两个有序序列的交集（仅保留两个范围共有的元素）。

语法

cpp

```cpp
template<class InputIterator1, class InputIterator2, class OutputIterator>
OutputIterator set_intersection(InputIterator1 first1, InputIterator1 last1,
                                InputIterator2 first2, InputIterator2 last2,
                                OutputIterator result);
```

示例

cpp

```cpp
#include <iostream>
#include <algorithm>
#include <vector>

int main() {
    std::vector<int> v1 = {1, 2, 3, 4};
    std::vector<int> v2 = {3, 4, 5, 6};
    std::vector<int> result;

    std::set_intersection(v1.begin(), v1.end(), v2.begin(), v2.end(),
                          std::back_inserter(result));

    for (int x : result)
        std::cout << x << " ";
    std::cout << "\n";
}
```

**输出:** 3 4

特点

- 结果只包含两个范围中都出现的元素。
- 时间复杂度: O(n + m)。

------

3. std::set_difference

作用

计算两个有序序列的差集（保留第一个范围中不在第二个范围中的元素）。

语法

cpp

```cpp
template<class InputIterator1, class InputIterator2, class OutputIterator>
OutputIterator set_difference(InputIterator1 first1, InputIterator1 last1,
                              InputIterator2 first2, InputIterator2 last2,
                              OutputIterator result);
```

示例

cpp

```cpp
#include <iostream>
#include <algorithm>
#include <vector>

int main() {
    std::vector<int> v1 = {1, 2, 3, 4};
    std::vector<int> v2 = {3, 4, 5, 6};
    std::vector<int> result;

    std::set_difference(v1.begin(), v1.end(), v2.begin(), v2.end(),
                        std::back_inserter(result));

    for (int x : result)
        std::cout << x << " ";
    std::cout << "\n";
}
```

**输出:** 1 2

特点

- 结果包含第一个范围中不在第二个范围中的元素。
- 时间复杂度: O(n + m)。

------

4. std::set_symmetric_difference

作用

计算两个有序序列的对称差集（保留只出现在一个范围中的元素，排除共有元素）。

语法

cpp

```cpp
template<class InputIterator1, class InputIterator2, class OutputIterator>
OutputIterator set_symmetric_difference(InputIterator1 first1, InputIterator1 last1,
                                        InputIterator2 first2, InputIterator2 last2,
                                        OutputIterator result);
```

示例

cpp

```cpp
#include <iostream>
#include <algorithm>
#include <vector>

int main() {
    std::vector<int> v1 = {1, 2, 3, 4};
    std::vector<int> v2 = {3, 4, 5, 6};
    std::vector<int> result;

    std::set_symmetric_difference(v1.begin(), v1.end(), v2.begin(), v2.end(),
                                  std::back_inserter(result));

    for (int x : result)
        std::cout << x << " ";
    std::cout << "\n";
}
```

**输出:** 1 2 5 6

特点

- 结果包含只出现在一个范围中的元素（即并集减去交集）。
- 时间复杂度: O(n + m)。

------

带比较器的版本

所有这些算法都有重载版本，允许传入自定义比较器：

cpp

```cpp
template<class InputIterator1, class InputIterator2, class OutputIterator, class Compare>
OutputIterator set_xxx(InputIterator1 first1, InputIterator1 last1,
                       InputIterator2 first2, InputIterator2 last2,
                       OutputIterator result, Compare comp);
```

- comp: 二元比较函数（如 std::greater<int> 用于降序）。

示例（降序）

cpp

```cpp
std::vector<int> v1 = {4, 3, 2, 1};
std::vector<int> v2 = {6, 5, 4, 3};
std::vector<int> result;

std::set_union(v1.begin(), v1.end(), v2.begin(), v2.end(),
               std::back_inserter(result), std::greater<int>());

for (int x : result)
    std::cout << x << " ";
std::cout << "\n";
```

**输出:** 6 5 4 3 2 1

------

对比总结

| 算法                          | 数学定义          | 输出内容                 |
| ----------------------------- | ----------------- | ------------------------ |
| std::set_union                | A ∪ B             | 所有元素（去重）         |
| std::set_intersection         | A ∩ B             | 共有元素                 |
| std::set_difference           | A - B             | A 中不在 B 的元素        |
| std::set_symmetric_difference | (A ∪ B) - (A ∩ B) | 只出现在一个集合中的元素 |

------

注意事项

1. **输入必须有序**: 如果输入范围未排序，结果未定义。
2. **输出空间**: result 必须指向足够大的空间，或者使用 std::back_inserter 动态扩展容器。
3. **迭代器要求**: 输入迭代器至少是 InputIterator，输出迭代器至少是 OutputIterator。
4. **性能**: 线性时间复杂度，适合有序数据。如果数据未排序，需先用 std::sort（增加 O(n log n) 开销）。

------

综合示例

cpp

```cpp
#include <iostream>
#include <algorithm>
#include <vector>

int main() {
    std::vector<int> v1 = {1, 2, 3, 4};
    std::vector<int> v2 = {3, 4, 5, 6};
    std::vector<int> result;

    std::cout << "Union: ";
    std::set_union(v1.begin(), v1.end(), v2.begin(), v2.end(),
                   std::back_inserter(result));
    for (int x : result) std::cout << x << " ";
    std::cout << "\n";
    result.clear();

    std::cout << "Intersection: ";
    std::set_intersection(v1.begin(), v1.end(), v2.begin(), v2.end(),
                          std::back_inserter(result));
    for (int x : result) std::cout << x << " ";
    std::cout << "\n";
    result.clear();

    std::cout << "Difference: ";
    std::set_difference(v1.begin(), v1.end(), v2.begin(), v2.end(),
                        std::back_inserter(result));
    for (int x : result) std::cout << x << " ";
    std::cout << "\n";
    result.clear();

    std::cout << "Symmetric Difference: ";
    std::set_symmetric_difference(v1.begin(), v1.end(), v2.begin(), v2.end(),
                                  std::back_inserter(result));
    for (int x : result) std::cout << x << " ";
    std::cout << "\n";
}
```

**输出:**

```text
Union: 1 2 3 4 5 6
Intersection: 3 4
Difference: 1 2
Symmetric Difference: 1 2 5 6
```

如果你有具体问题或需要更复杂示例，请告诉我！



## 例子2: 定义自己的比较器

```c++
#include <string>
#include <algorithm>
#include <vector>
#include <iostream>

struct E {
    int v;
    std::string label;
    auto operator<=>(const E& other) const {
        return v <=> other.v;
    }
};

int main() {
    std::vector<E> in1{{1,"a"},{2,"a-1"},{2,"a-2"}};
    std::vector<E> in2{{2,"b-1"},{2,"b-2"},{2,"b-3"},{3,"b"}};
    std::vector<E> out;

    // Intersection, common elements are picked from the first range
    std::set_intersection(in1.begin(), in1.end(),
        in2.begin(), in2.end(),
        std::back_inserter(out));
    // out == {{2,"a-1"},{2,"a-1"}}

    std::cout << "Intersect:\n";
    for (auto v : out)
        std::cout << v.v << " " << v.label << "\n";
    
    out.clear();
    // Union, overlapping elements are picked from the first range,
    // non-overlapping elements are picked from their source range.
    std::set_union(in1.begin(), in1.end(),
        in2.begin(), in2.end(),
        std::back_inserter(out));
    // out == {{1,"a"},{2,"a-1"},{2,"a-1"},{2,"b-3"},{3,"b"}}

    std::cout << "Union:\n";
    for (auto v : out)
        std::cout << v.v << " " << v.label << "\n";

    out.clear();
    // Difference, overlapping elements are skipped,
    // non-overlapping elements are picked from the first range.
    std::set_difference(in1.begin(), in1.end(),
        in2.begin(), in2.end(),
        std::back_inserter(out));
    // out == {{1,"a"}}

    std::cout << "Difference:\n";
    for (auto v : out)
        std::cout << v.v << " " << v.label << "\n";

    out.clear();
    // Symmetric difference, overlapping elements are skipped,
    // non-overlapping elements are picked from their source range.
    std::set_symmetric_difference(in1.begin(), in1.end(),
        in2.begin(), in2.end(),
        std::back_inserter(out));
    // out == {{1,"a"},{2,"b-3"},{3,"b"}}

    std::cout << "Symmetric:\n";
    for (auto v : out)
        std::cout << v.v << " " << v.label << "\n";
}
```

