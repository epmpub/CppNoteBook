# std::ranges::find_last std::ranges::find_last_if

这段代码展示了 C++20 Ranges 库中 std::ranges::find_last 和 std::ranges::find_last_if 的用法，结合 std::forward_list 和 std::views::counted。这些算法用于查找范围内最后一个满足条件的元素，并返回迭代器对（找到的元素和结束位置）。

------

完整代码

```cpp
#include <forward_list>
#include <algorithm>
#include <ranges>
#include <iostream>

int main() {
    std::forward_list<int> data{1, 2, 3, 4, 5, 6, 7};
    {
        auto [it, end] = std::ranges::find_last(data, 5);
        // *it == 5, end == data.end()
        std::cout << "*it == " << *it << "\n";
    }

    auto is_even = [](int v) { return v % 2 == 0; };
    {
        auto [it, end] = std::ranges::find_last_if(data, is_even);
        // *it == 6, end == data.end()
        std::cout << "*it == " << *it << "\n";
    }

    // the calculated end is useful when working with lazy ranges
    {
        auto counted = std::views::counted(data.begin(), 5);
        auto [it, end] = std::ranges::find_last_if(counted, is_even);
        // *it == 4, *end == 6
        std::cout << "*it == " << *it << "\n";
        std::cout << "*end == " << *end << "\n";    
    }
}
```

------

逐步解释

**初始化**

cpp

```cpp
std::forward_list<int> data{1, 2, 3, 4, 5, 6, 7};
```

- **data** 是一个单向链表（std::forward_list），元素为 {1, 2, 3, 4, 5, 6, 7}。
- 单向链表只支持从头到尾的遍历，std::ranges::find_last 内部会遍历整个范围以找到最后一个匹配项。

------

**部分 1：std::ranges::find_last**

cpp

```cpp
{
    auto [it, end] = std::ranges::find_last(data, 5);
    std::cout << "*it == " << *it << "\n";
}
```

- **std::ranges::find_last(data, 5)**：

  - 在 data 中查找最后一个等于 5 的元素。
  - 返回一个 subrange 结构，包含两个迭代器：
    - it：指向最后一个匹配元素的迭代器。
    - end：范围的结束迭代器（这里是 data.end()）。

- **执行过程**：

  - 遍历 {1, 2, 3, 4, 5, 6, 7}，找到最后一个 5（位置 4）。
  - it 指向 5，end 是 data.end()（链表末尾）。

- **输出**：

  ```text
  *it == 5
  ```

------

**部分 2：std::ranges::find_last_if**

cpp

```cpp
auto is_even = [](int v) { return v % 2 == 0; };
{
    auto [it, end] = std::ranges::find_last_if(data, is_even);
    std::cout << "*it == " << *it << "\n";
}
```

- **is_even**：

  - 一个 lambda 函数，判断数字是否为偶数。

- **std::ranges::find_last_if(data, is_even)**：

  - 在 data 中查找最后一个满足 is_even 的元素。
  - 返回结构同上：it 指向匹配元素，end 是范围结束。

- **执行过程**：

  - 遍历 {1, 2, 3, 4, 5, 6, 7}：
    - 偶数有 2（位置 1）、4（位置 3）、6（位置 5）。
    - 最后一个偶数是 6。
  - it 指向 6，end 是 data.end()。

- **输出**：

  ```text
  *it == 6
  ```

------

**部分 3：结合 std::views::counted**

cpp

```cpp
{
    auto counted = std::views::counted(data.begin(), 5);
    auto [it, end] = std::ranges::find_last_if(counted, is_even);
    std::cout << "*it == " << *it << "\n";
    std::cout << "*end == " << *end << "\n";    
}
```

- **std::views::counted(data.begin(), 5)**：

  - 创建一个视图，只包含 data 的前 5 个元素：{1, 2, 3, 4, 5}。
  - 返回类型是 std::ranges::subrange，范围从 data.begin() 到 data.begin() + 5。

- **std::ranges::find_last_if(counted, is_even)**：

  - 在 counted 视图中查找最后一个偶数。
  - 返回 it（匹配元素）和 end（视图的结束迭代器）。

- **执行过程**：

  - 遍历 {1, 2, 3, 4, 5}：
    - 偶数有 2（位置 1）、4（位置 3）。
    - 最后一个偶数是 4。
  - it 指向 4。
  - end 指向 counted 的结束位置，即 data.begin() + 5，对应 6。

- **输出**：

  ```text
  *it == 4
  *end == 6
  ```

------

输出总结

运行代码后，完整输出为：

```text
*it == 5
*it == 6
*it == 4
*end == 6
```

------

关键点分析

1. **std::ranges::find_last 和 find_last_if**：
   - find_last(data, value)：查找最后一个等于 value 的元素。
   - find_last_if(data, pred)：查找最后一个满足谓词 pred 的元素。
   - 返回 {iterator, sentinel} 对，iterator 是匹配位置，sentinel 是范围结束。
2. **std::views::counted**：
   - 限制范围为前 n 个元素，生成惰性视图。
   - 在本例中，counted 是 {1, 2, 3, 4, 5}，结束于 6。
3. **std::forward_list**：
   - 单向链表，不支持反向迭代，find_last 内部线性扫描到末尾。

------

时间复杂度

- **std::ranges::find_last 和 find_last_if**：
  - O(n)，n 是范围大小，需遍历整个范围。
- **std::views::counted**：
  - 构造：O(1)。
  - 迭代：O(k)，k 是指定计数（这里是 5）。

------

使用场景

1. **查找最后一个匹配项**：
   - 如日志处理中查找最后的事件。
2. **结合惰性视图**：
   - counted 示例展示了如何在子范围内查找。
3. **调试**：
   - 返回的 end 可验证范围边界。

------

注意事项

1. **C++20 要求**：
   - 需要 -std=c++20 和支持 Ranges 的编译器。
2. **迭代器有效性**：
   - it 和 end 是单向链表迭代器，需确保不越界。
3. **空范围**：
   - 若范围为空，it == end。

------

总结

这段代码展示了 std::ranges::find_last 和 find_last_if 的用法：

- **部分 1**：查找最后一个 5，结果是 5。
- **部分 2**：查找最后一个偶数，结果是 6。
- **部分 3**：在限定前 5 个元素的视图中查找最后一个偶数，结果是 4，结束于 6。 这些算法与 Ranges 视图（如 counted）结合，提供了灵活的查找能力。如果你有具体问题或想扩展代码，请告诉我！

在 C++20 的 Ranges 库中，std::ranges::find_last 和其变体（如 std::ranges::find_last_if）用于查找范围内最后一个匹配的元素。如果没有找到匹配的元素，它们的行为是明确定义的。下面详细解释 std::ranges::find_last 在未找到匹配元素时的返回值。

------

定义

cpp

```cpp
#include <algorithm>

namespace std::ranges {
    template<forward_range R, class T, class Proj = std::identity>
    requires std::indirect_binary_predicate<
        std::ranges::equal_to, projected<iterator_t<R>, Proj>, const T*>
    constexpr subrange<iterator_t<R>> find_last(R&& r, const T& value, Proj proj = {});
}
```

- **R**: 输入范围（需满足 forward_range，即支持正向迭代）。
- **T**: 要查找的值类型。
- **Proj**: 投影函数（默认为 std::identity）。
- 返回类型：std::ranges::subrange<iterator_t<R>>，包含两个迭代器：{iterator, sentinel}。

------

## 未找到匹配元素时的返回值

------

根据 C++20 标准（[alg.find.last]），当 std::ranges::find_last 在范围中没有找到匹配的元素时：

- **返回**：一个 std::ranges::subrange，其中：
  - **iterator（即 it）**：等于范围的结束迭代器（ranges::end(r)）。
  - **sentinel（即 end）**：也等于范围的结束迭代器（ranges::end(r)）。
- **含义**：
  - it == end，表示没有找到匹配项，迭代器指向范围末尾。

这与传统的 std::find 返回 end() 的行为类似，但 find_last 返回的是一个 subrange，提供了额外的结束信息。

------

示例代码

示例 1：未找到元素

cpp

```cpp
#include <ranges>
#include <forward_list>
#include <iostream>

int main() {
    std::forward_list<int> data{1, 2, 3, 4};

    auto [it, end] = std::ranges::find_last(data, 5);

    if (it == end) {
        std::cout << "Element 5 not found\n";
    } else {
        std::cout << "Found: " << *it << "\n";
    }

    return 0;
}
```

输出

```text
Element 5 not found
```

- **解释**：
  - data 是 {1, 2, 3, 4}，不包含 5。
  - std::ranges::find_last 返回 {data.end(), data.end()}。
  - it == end，表示未找到。

------

示例 2：空范围

cpp

```cpp
#include <ranges>
#include <forward_list>
#include <iostream>

int main() {
    std::forward_list<int> data; // 空范围

    auto [it, end] = std::ranges::find_last(data, 42);

    if (it == end) {
        std::cout << "Empty range or not found\n";
    } else {
        std::cout << "Found: " << *it << "\n";
    }

    return 0;
}
```

输出

```text
Empty range or not found
```

- **解释**：
  - 空范围中无元素，返回 {data.end(), data.end()}。
  - it == end。

------

std::ranges::find_last_if 的行为

对于 std::ranges::find_last_if（基于谓词查找），未找到匹配元素时行为相同：

- 返回 {ranges::end(r), ranges::end(r)}。

示例

cpp

```cpp
#include <ranges>
#include <forward_list>
#include <iostream>

int main() {
    std::forward_list<int> data{1, 3, 5}; // 全奇数
    auto is_even = [](int v) { return v % 2 == 0; };

    auto [it, end] = std::ranges::find_last_if(data, is_even);

    if (it == end) {
        std::cout << "No even number found\n";
    } else {
        std::cout << "Found: " << *it << "\n";
    }

    return 0;
}
```

输出

```text
No even number found
```

- **解释**：
  - 无偶数，返回 {data.end(), data.end()}。

------

返回类型

- **std::ranges::subrange<I>**：
  - I 是范围的迭代器类型。
  - 包含两个成员：iterator 和 sentinel。
- 未找到时：{end, end}。

------

使用建议

- **检查未找到**：

  - 总是比较 it == end 来判断是否找到元素。

  cpp

  ```cpp
  auto [it, end] = std::ranges::find_last(data, value);
  if (it != end) {
      // 找到元素
  } else {
      // 未找到
  }
  ```

- **避免解引用**：

  - 如果 it == end，解引用 it 是未定义行为。

------

时间复杂度

- **O(n)**：
  - n 是范围大小。
  - 需要遍历整个范围以确认最后一个匹配。

------

总结

std::ranges::find_last 如果没有找到匹配的元素，返回一个 std::ranges::subrange，其中 iterator 和 sentinel 都等于范围的结束迭代器（ranges::end(r)）。这适用于值查找（find_last）和谓词查找（find_last_if），包括空范围情况。使用时需检查 it == end 以避免未定义行为。如果你有具体场景或疑问，请告诉我！