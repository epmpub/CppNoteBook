#  std::search_n（<algorithm>）和 C++20 的 std::ranges::search_n（<ranges>）

```c++
#include <algorithm>
#include <ranges>
#include <cmath>
#include <vector>
#include <iostream>

int main() {
    std::vector<int> data{1,7,3,3,9,5,5,5,6,2};

    // First instance of three consecutive '5'
    auto it = std::search_n(data.begin(), data.end(), 3, 5);
    // std::views::counted(it, 3) == {5, 5, 5}

    for (int v : std::views::counted(it, 3))
        std::cout << v << " ";
    std::cout << "\n";

    // Range version returns a range
    auto rng = std::ranges::search_n(data, 2, 3);
    // rng == {3, 3}
    
    for (int v : rng)
        std::cout << v << " ";
    std::cout << "\n";

    // Comparator can be customized
    std::vector<double> approximate{2.4, 2.9, 3.0, 3.8, 3.1};
    auto floor = std::ranges::search_n(approximate, 3, 3.0,
        [](double a, double b) {
            return std::floor(a) == std::floor(b);
        });
    // odd == {3.0, 3.8, 3.1}

    for (double v : floor)
        std::cout << v << " ";
    std::cout << "\n";
}
```

这段代码展示了 C++ 中查找连续元素序列的两种算法：传统的 std::search_n（<algorithm>）和 C++20 的 std::ranges::search_n（<ranges>）。代码通过示例展示了它们的用法，包括默认比较和自定义比较器。以下是逐步解释。

------

代码概览

- 使用 std::search_n 查找向量中连续 3 个 5 的第一个位置。
- 使用 std::ranges::search_n 查找连续 2 个 3 的子范围。
- 使用自定义比较器查找连续 3 个接近 3.0 的浮点数。

------

关键组件

1. **头文件**

cpp

```cpp
#include <algorithm>
#include <ranges>
#include <cmath>
#include <vector>
#include <iostream>
```

- <algorithm>：提供 std::search_n。
- <ranges>：提供 std::ranges::search_n 和 std::views::counted。
- <cmath>：提供 std::floor。
- <vector>：提供 std::vector。
- <iostream>：用于输出。
- **std::search_n 示例**

cpp

```cpp
std::vector<int> data{1,7,3,3,9,5,5,5,6,2};
auto it = std::search_n(data.begin(), data.end(), 3, 5);
for (int v : std::views::counted(it, 3))
    std::cout << v << " ";
std::cout << "\n";
```

- **data**：
  - 向量内容：{1, 7, 3, 3, 9, 5, 5, 5, 6, 2}。
- **std::search_n**：
  - 原型：search_n(first, last, count, value)。
  - 在 [first, last) 中查找连续 count 个等于 value 的子序列。
  - 返回第一个匹配的起始迭代器，或 last（未找到）。
- **调用**：
  - std::search_n(data.begin(), data.end(), 3, 5)：
    - 查找连续 3 个 5。
    - 找到位置 5（{5, 5, 5}），it 指向 data[5]。
- **std::views::counted(it, 3)**：
  - 创建从 it 开始的长度为 3 的范围视图：{5, 5, 5}。
- **输出**：
  - 5 5 5。
- **std::ranges::search_n 示例**

cpp

```cpp
auto rng = std::ranges::search_n(data, 2, 3);
for (int v : rng)
    std::cout << v << " ";
std::cout << "\n";
```

- **std::ranges::search_n**：
  - 原型：search_n(range, count, value)。
  - 返回一个子范围（subrange），表示连续 count 个等于 value 的序列。
- **调用**：
  - std::ranges::search_n(data, 2, 3)：
    - 查找连续 2 个 3。
    - 找到位置 2（{3, 3}），rng 表示该子范围。
- **输出**：
  - 3 3。
- **自定义比较器示例**

cpp

```cpp
std::vector<double> approximate{2.4, 2.9, 3.0, 3.8, 3.1};
auto floor = std::ranges::search_n(approximate, 3, 3.0,
    [](double a, double b) {
        return std::floor(a) == std::floor(b);
    });
for (double v : floor)
    std::cout << v << " ";
std::cout << "\n";
```

- **approximate**：
  - 向量内容：{2.4, 2.9, 3.0, 3.8, 3.1}。
- **std::ranges::search_n**：
  - 原型扩展：search_n(range, count, value, pred)。
  - 使用谓词 pred 判断元素是否匹配 value。
- **调用**：
  - std::ranges::search_n(approximate, 3, 3.0, ...)：
    - 查找连续 3 个元素，其 floor 值等于 floor(3.0)（即 3）。
    - 比较器：std::floor(a) == std::floor(b)。
    - 检查：
      - floor(2.4) = 2，不匹配。
      - floor(2.9) = 2，不匹配。
      - floor(3.0) = 3，匹配。
      - floor(3.8) = 3，匹配。
      - floor(3.1) = 3，匹配。
    - 找到位置 2（{3.0, 3.8, 3.1}）。
- **输出**：
  - 3 3.8 3.1。

------

为什么这样工作？

1. **std::search_n**：
   - 返回迭代器，需手动构造范围。
   - 使用 == 比较。
2. **std::ranges::search_n**：
   - 返回子范围，更现代化。
   - 支持自定义比较器。
3. **Ranges 库**：
   - counted 创建固定长度视图。
   - subrange 表示匹配结果。
4. **自定义比较**：
   - 灵活匹配（如基于 floor）。

------

输出

```text
5 5 5
3 3
3 3.8 3.1
```

------

使用场景

- **模式匹配**：
  - 查找连续重复的元素。
- **数据分析**：
  - 检测特定序列。
- **自定义规则**：
  - 使用谓词处理复杂匹配。

------

总结

- std::search_n 查找连续 3 个 5，返回迭代器。
- std::ranges::search_n 查找连续 2 个 3，返回范围。
- 自定义比较器查找连续 3 个接近 3.0 的值。
- 代码展示了传统和 Ranges 算法的用法及灵活性。