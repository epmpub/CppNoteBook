

#  C++  std::partial_sort（<algorithm>） C++20 std::ranges::partial_sort（<ranges>）

代码:

```C++
#include <vector>
#include <algorithm>
#include <iostream>

int main() {
    std::vector<int> data{3,9,8,5,1,2,7,4,6};
    
    for (auto v : data)
        std::cout << v << " ";
    std::cout << "\n";

    // partially sort, so that the first three elements are sorted
    std::partial_sort(data.begin(), data.begin()+3, data.end());
    // data == {1, 2, 3, ...}

    for (auto v : data)
        std::cout << v << " ";
    std::cout << "\n";

    // range version with a custom comparator
    std::ranges::partial_sort(data, data.begin()+3, std::greater<>{});
    // data == {9, 8, 7, ...}

    for (auto v : data)
        std::cout << v << " ";
    std::cout << "\n";

    struct V { int v; };
    std::vector<V> nested{{3},{9},{8},{5},{1},{2},{7},{4},{6}};

    // projecting each element to the v member
    std::ranges::partial_sort(nested, nested.begin()+3, std::less<>{}, &V::v);

    for (auto v : nested)
        std::cout << v.v << " ";
    std::cout << "\n";
}
```

这段代码展示了 C++ 中部分排序的两种算法：传统的 std::partial_sort（<algorithm>）和 C++20 的 std::ranges::partial_sort（<ranges>）。代码通过示例展示了它们的用法，包括默认排序、自定义比较器和投影功能。以下是逐步解释。

------

代码概览

- 使用 std::partial_sort 将向量前 3 个元素排序。
- 使用 std::ranges::partial_sort 以降序排序前 3 个元素。
- 对包含结构体 V 的向量，按成员 v 部分排序。

------

关键组件

1. **头文件**

cpp

```cpp
#include <vector>
#include <algorithm>
#include <iostream>
```

- <vector>：提供 std::vector。
- <algorithm>：提供 std::partial_sort 和 std::ranges::partial_sort（通过 <ranges>）。
- <iostream>：用于输出。
- **std::partial_sort 示例**

cpp

```cpp
std::vector<int> data{3,9,8,5,1,2,7,4,6};
for (auto v : data)
    std::cout << v << " ";
std::cout << "\n";

std::partial_sort(data.begin(), data.begin()+3, data.end());
for (auto v : data)
    std::cout << v << " ";
std::cout << "\n";
```

- **data**：

  - 初始内容：{3, 9, 8, 5, 1, 2, 7, 4, 6}。

- **std::partial_sort**：

  - 原型：partial_sort(first, middle, last)。
  - 将 [first, last) 中的元素部分排序，确保 [first, middle) 是有序的（默认升序）。
  - 剩余部分（[middle, last)）无序。

- **调用**：

  - std::partial_sort(data.begin(), data.begin()+3, data.end())：
    - 前 3 个元素排序为 {1, 2, 3}。
    - 剩余元素位置不定，例如 {9, 8, 5, 7, 4, 6}。

- **结果**：

  - data == {1, 2, 3, 9, 8, 5, 7, 4, 6}（具体后半部分依赖实现）。

- **输出**：

  ```text
  3 9 8 5 1 2 7 4 6
  1 2 3 9 8 5 7 4 6
  ```

- **std::ranges::partial_sort 与自定义比较器**

cpp

```cpp
std::ranges::partial_sort(data, data.begin()+3, std::greater<>{});
for (auto v : data)
    std::cout << v << " ";
std::cout << "\n";
```

- **std::ranges::partial_sort**：

  - 原型：partial_sort(range, middle, comp)。
  - 对 range 部分排序，直到 middle，使用比较器 comp。

- **调用**：

  - std::ranges::partial_sort(data, data.begin()+3, std::greater<>{})：
    - std::greater<>{} 表示降序。
    - 前 3 个元素排序为 {9, 8, 7}。
    - 剩余部分无序，例如 {1, 2, 3, 5, 4, 6}。

- **结果**：

  - data == {9, 8, 7, 1, 2, 3, 5, 4, 6}（后半部分依赖实现）。

- **输出**：

  ```text
  9 8 7 1 2 3 5 4 6
  ```

- **投影与结构体**

cpp

```cpp
struct V { int v; };
std::vector<V> nested{{3},{9},{8},{5},{1},{2},{7},{4},{6}};
std::ranges::partial_sort(nested, nested.begin()+3, std::less<>{}, &V::v);
for (auto v : nested)
    std::cout << v.v << " ";
std::cout << "\n";
```

- **nested**：

  - 初始内容：{{3}, {9}, {8}, {5}, {1}, {2}, {7}, {4}, {6}}。

- **std::ranges::partial_sort**：

  - 原型扩展：partial_sort(range, middle, comp, proj)。
  - 使用投影函数 proj 提取比较值。

- **调用**：

  - std::ranges::partial_sort(nested, nested.begin()+3, std::less<>{}, &V::v)：
    - std::less<>{} 表示升序。
    - 投影 &V::v 提取成员 v。
    - 前 3 个元素按 v 排序为 {{1}, {2}, {3}}。
    - 剩余部分无序，例如 {{9}, {8}, {5}, {7}, {4}, {6}}。

- **结果**：

  - nested == {{1}, {2}, {3}, {9}, {8}, {5}, {7}, {4}, {6}}（后半部分依赖实现）。

- **输出**：

  ```text
  1 2 3 9 8 5 7 4 6
  ```

------

为什么这样工作？

1. **std::partial_sort**：
   - 使用堆排序（或类似算法），仅排序前 middle - first 个最小元素。
   - 时间复杂度：O(n log k)，其中 k = middle - first。
2. **std::ranges::partial_sort**：
   - 更现代，支持范围、比较器和投影。
   - 同样部分排序，但接口更灵活。
3. **投影**：
   - &V::v 将比较限制在成员 v，不影响整个对象。

------

输出

```text
3 9 8 5 1 2 7 4 6
1 2 3 9 8 5 7 4 6
9 8 7 1 2 3 5 4 6
1 2 3 9 8 5 7 4 6
```

- 后半部分的具体顺序可能因实现而异。

------

使用场景

- **优先级排序**：
  - 只需前几个元素有序。
- **性能优化**：
  - 避免完整排序的开销。
- **复杂类型**：
  - 使用投影处理结构体字段。

------

总结

- std::partial_sort 将前 3 个元素升序排序为 {1, 2, 3}。
- std::ranges::partial_sort 以降序排序为 {9, 8, 7}。
- 投影版本按 V::v 排序为 {1, 2, 3}。
- 代码展示了部分排序的灵活性和应用。
- 