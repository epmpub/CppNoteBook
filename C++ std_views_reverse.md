# std::views::reverse

std::views::reverse 是 C++20 引入的一个 Ranges 视图适配器，定义在 <ranges> 头文件中。它用于创建一个反转视图，将输入范围的元素按逆序访问，而不修改原始数据。std::views::reverse 是惰性求值的，仅在迭代时按需计算，非常适合需要反向遍历的场景。

以下是对 std::views::reverse 的详细解释：

------



```cpp
#include <ranges>

namespace std::views {
    inline constexpr auto reverse = /* unspecified */;
}
```

- **reverse**：

  - 生成输入范围的反转视图。

- **语法**：

  cpp

  ```cpp
  auto view = std::views::reverse(range);
  ```

  - **range**: 输入范围，需满足 std::ranges::bidirectional_range。

- 返回类型：std::ranges::reverse_view<R>。

------

行为

- **std::views::reverse**：
  - 创建一个视图，其中元素顺序与原始范围相反。
  - 首元素变成末元素，末元素变成首元素。
- **惰性求值**：
  - 不复制数据，仅调整迭代顺序。
- **元素类型**：
  - 与原始范围的元素类型相同（引用类型保持不变）。

------

前提条件

- **范围要求**：
  - 输入范围必须满足 std::ranges::bidirectional_range。
  - 即需要支持双向迭代器（begin() 和 end() 可向前后移动）。
  - 如 std::vector、std::list、std::deque 适用，std::forward_list 不适用（单向）。
- **元素访问**：
  - 元素必须可通过迭代器访问。

------

示例代码

示例 1：基本用法

cpp

```cpp
#include <ranges>
#include <vector>
#include <iostream>

int main() {
    std::vector<int> vec = {1, 2, 3, 4, 5};

    auto reversed = std::views::reverse(vec);

    std::cout << "Reversed: ";
    for (int x : reversed) {
        std::cout << x << " ";
    }
    std::cout << "\n";

    std::cout << "Original: ";
    for (int x : vec) {
        std::cout << x << " ";
    }
    std::cout << "\n";

    return 0;
}
```

输出

```text
Reversed: 5 4 3 2 1
Original: 1 2 3 4 5
```

- **解释**：
  - reversed 是 vec 的反转视图：{5, 4, 3, 2, 1}。
  - 原始 vec 不变。

示例 2：与管道操作符结合

cpp

```cpp
#include <ranges>
#include <vector>
#include <iostream>

int main() {
    std::vector<int> vec = {1, 2, 3, 4, 5};

    auto even_reversed = vec | std::views::reverse | std::views::filter([](int x) { return x % 2 == 0; });

    std::cout << "Even reversed: ";
    for (int x : even_reversed) {
        std::cout << x << " ";
    }
    std::cout << "\n";

    return 0;
}
```

输出

```text
Even reversed: 4 2
```

- **解释**：
  - 先反转：{5, 4, 3, 2, 1}。
  - 再过滤偶数：{4, 2}。

------

返回视图的类型

- **std::ranges::reverse_view<R>**：
  - R 是输入范围类型。
- **元素类型**：
  - std::ranges::range_reference_t<R>（原始范围的引用类型）。

------

时间复杂度

- **构造**：O(1)，视图是惰性的。
- **迭代**：
  - 每次访问：O(1)（双向迭代器支持）。
  - 总复杂度：O(n)，n 是范围大小。

------

使用场景

1. **反向遍历**：

   - 无需修改原始数据反向访问。

   cpp

   ```cpp
   auto rev = std::views::reverse(vec);
   ```

2. **视图组合**：

   - 与 filter、transform 等结合处理。

3. **调试**：

   - 检查数据从尾到头的顺序。

------

注意事项

1. **C++20 要求**：
   - 需要 -std=c++20 和支持 Ranges 的编译器。
2. **双向迭代器**：
   - 输入范围必须支持双向迭代，否则编译失败。
   - std::forward_list 不支持。
3. **原始数据不变**：
   - 视图仅调整访问顺序，不影响底层容器。

------

与 std::reverse 的对比

| 特性       | std::views::reverse  | std::reverse     |
| ---------- | -------------------- | ---------------- |
| 引入版本   | C++20                | C++98            |
| 类型       | 视图适配器（惰性）   | 算法（立即执行） |
| 修改数据   | 否                   | 是               |
| 返回       | 反转视图             | 无（void）       |
| 迭代器要求 | 双向迭代器           | 双向迭代器       |
| 时间复杂度 | 构造 O(1)，迭代 O(n) | O(n)             |

示例对比

cpp

```cpp
#include <algorithm>
#include <ranges>
#include <vector>
#include <iostream>

int main() {
    std::vector<int> vec1 = {1, 2, 3};
    std::vector<int> vec2 = {1, 2, 3};

    auto rev_view = std::views::reverse(vec1);
    std::reverse(vec2.begin(), vec2.end());

    std::cout << "View: ";
    for (int x : rev_view) std::cout << x << " ";
    std::cout << "\nOriginal vec1: ";
    for (int x : vec1) std::cout << x << " ";
    std::cout << "\nModified vec2: ";
    for (int x : vec2) std::cout << x << " ";
    std::cout << "\n";

    return 0;
}
```

输出

```text
View: 3 2 1
Original vec1: 1 2 3
Modified vec2: 3 2 1
```

------

替代方案（C++17 前）

使用迭代器手动反转：

cpp

```cpp
for (auto it = vec.rbegin(); it != vec.rend(); ++it) {
    // 处理 *it
}
```

- **缺点**：不如 Ranges 视图简洁。

------

总结

std::views::reverse 是 C++20 中一个高效的视图适配器：

- 创建范围的反转视图，元素顺序逆向访问。
- 不修改原始数据，支持与其他 Ranges 视图组合。
- 要求双向迭代器，时间复杂度为迭代时的 O(n)。 它是现代 C++ 中处理反向遍历的优雅工具，相比 std::reverse，更灵活且无副作用。如果你有具体问题或想探讨用法，请告诉我！