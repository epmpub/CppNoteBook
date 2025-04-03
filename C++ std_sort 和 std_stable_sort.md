# std::sort 和 std::stable_sort

在 C++ 中，std::sort 和 std::stable_sort 是标准库中用于排序的算法，定义在 <algorithm> 头文件中。它们都用于对容器中的元素进行排序，但有一个关键区别：std::sort 不保证稳定性，而 std::stable_sort 保证稳定性。以下是对两者的详细解释：

------

std::sort

定义

- **头文件**：<algorithm>

- **函数签名**：

  cpp

  ```cpp
  template<class RandomIt>
  void sort(RandomIt first, RandomIt last);
  
  template<class RandomIt, class Compare>
  void sort(RandomIt first, RandomIt last, Compare comp);
  ```

  - first, last：要排序的范围（随机访问迭代器）。
  - comp：可选的比较函数（默认使用 operator<）。

- **行为**：

  - 对范围 [first, last) 中的元素进行升序排序（默认）。
  - **不稳定排序**：相等元素的相对顺序可能改变。
  - 通常使用内省排序（Introsort），结合快速排序、堆排序和插入排序。

- **复杂度**：

  - 平均：O(n log n)。
  - 最坏：O(n log n)（由于内省机制）。

示例

cpp

```cpp
#include <iostream>
#include <vector>
#include <algorithm>

int main() {
    std::vector<int> vec = {3, 1, 4, 1, 5};
    std::sort(vec.begin(), vec.end());

    for (int x : vec) {
        std::cout << x << " ";
    }
    std::cout << "\n";
    return 0;
}
```

- **输出**：1 1 3 4 5
- **说明**：简单升序排序，两个 1 的相对顺序可能改变。

自定义比较器

cpp

```cpp
#include <iostream>
#include <vector>
#include <algorithm>

int main() {
    std::vector<int> vec = {3, 1, 4, 1, 5};
    std::sort(vec.begin(), vec.end(), std::greater<int>{}); // 降序

    for (int x : vec) {
        std::cout << x << " ";
    }
    std::cout << "\n";
    return 0;
}
```

- **输出**：5 4 3 1 1
- **说明**：使用 std::greater<int> 实现降序排序。

------

std::stable_sort

定义

- **头文件**：<algorithm>

- **函数签名**：

  cpp

  ```cpp
  template<class RandomIt>
  void stable_sort(RandomIt first, RandomIt last);
  
  template<class RandomIt, class Compare>
  void stable_sort(RandomIt first, RandomIt last, Compare comp);
  ```

  - 参数与 std::sort 相同。

- **行为**：

  - 对范围 [first, last) 中的元素进行升序排序（默认）。
  - **稳定排序**：相等元素的相对顺序保持不变。
  - 通常使用归并排序（Merge Sort）或其变种实现。

- **复杂度**：

  - 平均：O(n log n)。
  - 最坏：O(n log n)，但可能需要额外的 O(n) 空间。

示例

cpp

```cpp
#include <iostream>
#include <vector>
#include <algorithm>

struct Item {
    int value;
    int index;
    Item(int v, int i) : value(v), index(i) {}
};

int main() {
    std::vector<Item> vec = {{2, 0}, {1, 1}, {2, 2}, {1, 3}};
    std::stable_sort(vec.begin(), vec.end(), 
                     [](const Item& a, const Item& b) { return a.value < b.value; });

    for (const auto& item : vec) {
        std::cout << "{" << item.value << ", " << item.index << "} ";
    }
    std::cout << "\n";
    return 0;
}
```

- **输出**：{1, 1} {1, 3} {2, 0} {2, 2}
- **说明**：
  - 按 value 排序。
  - 两个 value=1 和两个 value=2 的相对顺序（index）保持不变。

------

稳定性示例对比

cpp

```cpp
#include <iostream>
#include <vector>
#include <algorithm>

struct Item {
    int value;
    int index;
    Item(int v, int i) : value(v), index(i) {}
};

int main() {
    std::vector<Item> vec1 = {{2, 0}, {1, 1}, {2, 2}, {1, 3}};
    std::vector<Item> vec2 = vec1;

    // std::sort（不稳定）
    std::sort(vec1.begin(), vec1.end(), 
              [](const Item& a, const Item& b) { return a.value < b.value; });
    std::cout << "std::sort: ";
    for (const auto& item : vec1) {
        std::cout << "{" << item.value << ", " << item.index << "} ";
    }
    std::cout << "\n";

    // std::stable_sort（稳定）
    std::stable_sort(vec2.begin(), vec2.end(), 
                     [](const Item& a, const Item& b) { return a.value < b.value; });
    std::cout << "std::stable_sort: ";
    for (const auto& item : vec2) {
        std::cout << "{" << item.value << ", " << item.index << "} ";
    }
    std::cout << "\n";

    return 0;
}
```

- **输出示例**：

  ```text
  std::sort: {1, 3} {1, 1} {2, 2} {2, 0}
  std::stable_sort: {1, 1} {1, 3} {2, 0} {2, 2}
  ```

- **说明**：

  - std::sort：value=1 和 value=2 的相对顺序可能颠倒。
  - std::stable_sort：原始顺序（index）保持不变。

------

主要区别

| 特性       | std::sort                    | std::stable_sort         |
| ---------- | ---------------------------- | ------------------------ |
| 稳定性     | 不稳定（相等元素顺序可能变） | 稳定（相等元素顺序不变） |
| 算法       | 内省排序（Introsort）        | 归并排序（Merge Sort）   |
| 空间复杂度 | O(1)                         | O(n)（可能需要额外空间） |
| 时间复杂度 | O(n log n)                   | O(n log n)               |
| 适用场景   | 性能优先，不关心顺序         | 需要保持相等元素顺序     |

------

并行支持（C++17）

两者都支持执行策略：

cpp

```cpp
#include <execution>
// ...
std::sort(std::execution::par, vec.begin(), vec.end());
std::stable_sort(std::execution::par, vec.begin(), vec.end());
```

- std::execution::par：并行执行。
- 性能提升依赖编译器和硬件支持。

------

注意事项

- **迭代器要求**：需要随机访问迭代器（如 std::vector 或数组）。
- **比较器要求**：必须满足严格弱序（无环、不自反、对称传递）。
- **异常**：如果比较器抛出异常，排序可能部分完成，容器状态未定义。
- **选择依据**：
  - 如果不需要稳定性，std::sort 通常更快。
  - 如果需要稳定性（如多字段排序），使用 std::stable_sort。

------

总结

- **std::sort**：高效的不稳定排序，适合大多数不需要保持相对顺序的场景。
- **std::stable_sort**：稳定的排序，适合需要保留相等元素顺序的场景（如多级排序）。 两者都是 C++ 标准库中强大的排序工具，选择取决于具体需求（性能 vs 稳定性）。