#  std::count 和 std::count_if

我来为你详细解释 C++ 标准库中的 std::count 和 std::count_if，它们都定义在 <algorithm> 头文件中，用于统计序列中符合条件的元素个数。

------

std::count

作用

std::count 用于统计一个范围中等于某个特定值的元素个数。

语法

cpp

```cpp
template<class InputIterator, class T>
typename iterator_traits<InputIterator>::difference_type
count(InputIterator first, InputIterator last, const T& value);
```

- first, last: 定义要检查的范围 [first, last) 的迭代器。
- value: 要统计的目标值。
- 返回值: 范围内等于 value 的元素个数，类型通常是 ptrdiff_t（由 iterator_traits 推导）。

示例

cpp

```cpp
#include <iostream>
#include <algorithm>
#include <vector>

int main() {
    std::vector<int> vec = {1, 2, 3, 2, 4, 2, 5};
    int value = 2;

    auto result = std::count(vec.begin(), vec.end(), value);

    std::cout << "值 " << value << " 出现了 " << result << " 次\n";

    return 0;
}
```

**输出:** 值 2 出现了 3 次

特点

- 使用 == 操作符比较元素和 value，因此元素类型必须支持 ==。
- 时间复杂度: O(n)，其中 n 是范围的长度。

------

std::count_if

作用

std::count_if 用于统计一个范围内满足某个条件的元素个数，条件由用户提供的谓词（predicate）定义。

语法

cpp

```cpp
template<class InputIterator, class UnaryPredicate>
typename iterator_traits<InputIterator>::difference_type
count_if(InputIterator first, InputIterator last, UnaryPredicate pred);
```

- first, last: 定义要检查的范围 [first, last) 的迭代器。
- pred: 一元谓词函数，接受一个元素并返回 bool 值（true 表示满足条件）。
- 返回值: 范围内满足 pred 条件的元素个数。

示例

cpp

```cpp
#include <iostream>
#include <algorithm>
#include <vector>

int main() {
    std::vector<int> vec = {1, 2, 3, 4, 5, 6};

    // 统计偶数的个数
    auto result = std::count_if(vec.begin(), vec.end(), 
                                [](int x) { return x % 2 == 0; });

    std::cout << "偶数个数: " << result << "\n";

    return 0;
}
```

**输出:** 偶数个数: 3

特点

- 提供了更大的灵活性，可以根据自定义条件统计。
- 谓词可以是函数、函数对象或 lambda 表达式。
- 时间复杂度: O(n)，其中 n 是范围的长度。

------

对比

| 特性         | std::count               | std::count_if                |
| ------------ | ------------------------ | ---------------------------- |
| 功能         | 统计等于特定值的元素个数 | 统计满足自定义条件的元素个数 |
| 参数         | 需要一个目标值 value     | 需要一个谓词 pred            |
| 比较方式     | 使用 == 操作符           | 使用谓词返回的 bool 值       |
| 灵活性       | 固定为相等比较           | 可自定义条件                 |
| 典型使用场景 | 统计特定值的出现次数     | 统计符合复杂条件的元素个数   |

------

结合使用的例子

cpp

```cpp
#include <iostream>
#include <algorithm>
#include <vector>

int main() {
    std::vector<int> vec = {1, 2, 3, 2, 4, 5, 2, 6};

    // 使用 std::count 统计值为 2 的个数
    auto count_2 = std::count(vec.begin(), vec.end(), 2);
    std::cout << "值为 2 的个数: " << count_2 << "\n";

    // 使用 std::count_if 统计大于 3 的个数
    auto count_gt_3 = std::count_if(vec.begin(), vec.end(), 
                                    [](int x) { return x > 3; });
    std::cout << "大于 3 的个数: " << count_gt_3 << "\n";

    return 0;
}
```

**输出:**

```text
值为 2 的个数: 3
大于 3 的个数: 3
```

------

注意事项

1. **迭代器要求**: 两者都要求至少是输入迭代器（InputIterator）。
2. **性能**: 都是线性时间复杂度，适合中小规模数据。对于大规模数据，可能需要更高效的数据结构（如哈希表）。
3. **类型安全性**: std::count 的 value 类型必须与范围元素类型兼容；std::count_if 的谓词必须能处理范围中的元素。

如果你有具体问题或需要更复杂的示例，请告诉我！