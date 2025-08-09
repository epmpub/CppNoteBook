# std::binary_search

我来把这个 C++ 代码的解释翻译成中文，并逐段说明。

cpp

```cpp
#include <algorithm>
#include <vector>
#include <iostream>
#include <ranges>

int main() {
    std::vector<int> data{1,2,3,4,5,6,7,8,9};

    // 使用默认的 operator< 作为比较器
    bool contains = std::binary_search(data.begin(), data.end(), 4);
    // contains == true

    std::cout << std::boolalpha << "contains == " << contains << "\n";

    // 部分有序的例子
    data = {3,1,2,4,8,9,7,6,5};
    // 可以工作，因为：
    // - {3,1,2} 都小于 4
    // - {8,9,7,6,5} 都不小于 4
    contains = std::binary_search(data.begin(), data.end(), 4);
    // contains == true

    std::cout << "contains == " << contains << "\n";

    // 自定义比较器示例
    std::ranges::sort(data, std::ranges::greater{});
    // 比较器必须与范围的排序顺序一致
    contains = std::ranges::binary_search(data, 42, std::ranges::greater{});
    // contains == false

    std::cout << "contains == " << contains << "\n";

    contains = std::ranges::binary_search(data, 4, std::ranges::greater{});
    // contains == true

    std::cout << "contains == " << contains << "\n";

    // 支持投影，但需要注意：
    // 使用自定义投影时，所需的排序适用于投影后的值
    contains = std::ranges::binary_search(data, -4, std::ranges::less{}, std::negate<>{});
    // contains == true
    // 对值取反将比较器从 greater 翻转为 less

    std::cout << "contains == " << contains << "\n";
}
```

解释：

1. **基本的二分查找（使用默认比较器）**：

cpp

```cpp
std::vector<int> data{1,2,3,4,5,6,7,8,9};
bool contains = std::binary_search(data.begin(), data.end(), 4);
```

- 使用标准的 std::binary_search，要求范围已排序
- 默认比较器是 operator<（小于）
- 在已排序的向量中查找 4
- 返回 true，因为 4 存在于序列中
- 时间复杂度：O(log n)

1. **部分有序的例子**：

cpp

```cpp
data = {3,1,2,4,8,9,7,6,5};
contains = std::binary_search(data.begin(), data.end(), 4);
```

- 尽管不是完全排序的，但仍然有效，因为：
  - 4 之前的元素 (3,1,2) 都小于 4
  - 4 之后的元素 (8,9,7,6,5) 都大于或等于 4
- 二分查找只要求元素相对于查找值是分区的
- 返回 true，因为 4 存在

1. **使用自定义比较器（std::ranges）**：

cpp

```cpp
std::ranges::sort(data, std::ranges::greater{});
contains = std::ranges::binary_search(data, 42, std::ranges::greater{});
```

- 使用 greater{} 比较器将数据按降序排序
- 排序后，data 变为 {9,8,7,6,5,4,3,2,1}
- 使用匹配的 greater{} 比较器查找 42
- 返回 false，因为 42 不存在
- 比较器必须与排序顺序一致

1. **自定义比较器查找已有值**：

cpp

```cpp
contains = std::ranges::binary_search(data, 4, std::ranges::greater{});
```

- 在同一个降序向量中查找 4
- 返回 true，因为 4 存在
- 使用与排序相同的 greater{} 比较器

1. **投影示例**：

cpp

```cpp
contains = std::ranges::binary_search(data, -4, std::ranges::less{}, std::negate<>{});
```

- 使用投影 (std::negate) 在比较前对值取反
- 数据 {9,8,7,6,5,4,3,2,1} 在比较时变为 {-9,-8,-7,-6,-5,-4,-3,-2,-1}
- 使用 less{} 比较器查找 -4
- 返回 true，因为 4 存在（取反后与 -4 匹配）
- 由于取反，less{} 比较器实际上等效于 greater{}

关键点：

- 二分查找要求范围相对于查找值是分区的

- 自定义比较器必须与排序顺序一致

- ### **投影在比较前转换值**

- 所有版本的时间复杂度均为 O(log n)

- C++20 的 ranges 版本在比较器和投影方面提供了更多灵活性

输出结果：

```text
contains == true
contains == true
contains == false
contains == true
contains == true
```