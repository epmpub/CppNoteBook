# std::views::keys 和 std::views::values 

在 C++ 中，std::views::keys 和 std::views::values 是 C++20 引入的 Ranges 库中的视图适配器，定义在 <ranges> 头文件中。它们用于从关联容器（如 std::map、std::unordered_map）或类似键值对的范围中提取键（keys）或值（values），生成对应的视图。这些视图是惰性求值的，仅在迭代时访问底层数据，非常适合处理键值对数据的特定部分。

以下是对 std::views::keys 和 std::views::values 的详细解释：

------

定义

cpp

```cpp
#include <ranges>

namespace std::views {
    inline constexpr auto keys = /* unspecified */;
    inline constexpr auto values = /* unspecified */;
}
```

- **keys**：生成一个视图，包含输入范围中每个元素的键（第一部分）。
- **values**：生成一个视图，包含输入范围中每个元素的值（第二部分）。
- 返回类型分别是 std::ranges::keys_view 和 std::ranges::values_view。

语法

cpp

```cpp
auto keys_view = std::views::keys(range);
auto values_view = std::views::values(range);
```

- **range**: 输入范围，需满足 std::ranges::input_range，且元素需支持键值对访问（通常是 std::pair 或类似结构）。

------

行为

- **std::views::keys**：
  - 从每个元素中提取 first 成员（键），生成只包含键的视图。
- **std::views::values**：
  - 从每个元素中提取 second 成员（值），生成只包含值的视图。
- **惰性求值**：
  - 视图不复制数据，仅提供对底层范围的引用。
- **元素类型**：
  - 键视图的元素类型是 decltype(range[0].first)。
  - 值视图的元素类型是 decltype(range[0].second)。

------

前提条件

- **范围要求**：
  - 输入范围必须是 std::ranges::input_range。
  - 元素必须是键值对类型，通常是 std::pair<Key, Value> 或类似结构（支持 .first 和 .second）。
- **典型适用场景**：
  - 关联容器（如 std::map、std::unordered_map）。
  - 自定义范围，只要元素提供键值对访问。

------

示例代码

示例 1：基本用法（std::map）

cpp

```cpp
#include <ranges>
#include <iostream>
#include <map>

int main() {
    std::map<int, std::string> m = {{1, "one"}, {2, "two"}, {3, "three"}};

    // 提取键
    auto keys = std::views::keys(m);
    std::cout << "Keys: ";
    for (int k : keys) {
        std::cout << k << " ";
    }
    std::cout << "\n";

    // 提取值
    auto values = std::views::values(m);
    std::cout << "Values: ";
    for (const auto& v : values) {
        std::cout << v << " ";
    }
    std::cout << "\n";

    return 0;
}
```

输出

```text
Keys: 1 2 3
Values: one two three
```

- **解释**：
  - keys 提取 {1, 2, 3}。
  - values 提取 {"one", "two", "three"}。

示例 2：与 std::vector<std::pair> 结合

cpp

```cpp
#include <ranges>
#include <iostream>
#include <vector>
#include <utility>

int main() {
    std::vector<std::pair<int, char>> pairs = {{1, 'a'}, {2, 'b'}, {3, 'c'}};

    auto keys = std::views::keys(pairs);
    std::cout << "Keys: ";
    for (int k : keys) {
        std::cout << k << " ";
    }
    std::cout << "\n";

    auto values = std::views::values(pairs);
    std::cout << "Values: ";
    for (char v : values) {
        std::cout << v << " ";
    }
    std::cout << "\n";

    return 0;
}
```

输出

```text
Keys: 1 2 3
Values: a b c
```

- **解释**：
  - pairs 是 std::pair 的向量，keys 和 values 分别提取 first 和 second。

示例 3：与管道操作符结合

cpp

```cpp
#include <ranges>
#include <iostream>
#include <map>

int main() {
    std::map<int, std::string> m = {{1, "one"}, {2, "two"}, {3, "three"}};

    // 使用管道操作符
    auto even_keys = m | std::views::keys | std::views::filter([](int k) { return k % 2 == 0; });

    std::cout << "Even keys: ";
    for (int k : even_keys) {
        std::cout << k << " ";
    }
    std::cout << "\n";

    return 0;
}
```

输出

```text
Even keys: 2
```

- **解释**：
  - 先提取键 {1, 2, 3}，再过滤出偶数 {2}。

------

返回视图的类型

- **std::ranges::keys_view<R>**：
  - 元素类型：std::ranges::range_reference_t<R>::first_type。
- **std::ranges::values_view<R>**：
  - 元素类型：std::ranges::range_reference_t<R>::second_type。
- **迭代器**：
  - 提供对键或值的直接访问。

------

时间复杂度

- **构造**：O(1)，视图是惰性生成的。
- **迭代**：
  - 每次访问：O(1)（仅引用底层元素）。
  - 总复杂度：O(n)，n 是范围大小。

------

使用场景

1. **键值分离**：

   - 从映射中单独处理键或值。

   cpp

   ```cpp
   auto keys = std::views::keys(map);
   ```

2. **数据分析**：

   - 统计键的属性或值的分布。

3. **与其他视图组合**：

   - 过滤、变换键或值。

   cpp

   ```cpp
   auto long_values = map | std::views::values | std::views::filter([](const auto& s) { return s.size() > 3; });
   ```

------

注意事项

1. **C++20 要求**：

   - 需要 -std=c++20 和支持 C++20 的编译器。

2. **输入范围**：

   - 必须是键值对范围，否则编译失败。
   - 不适用于普通序列（如 std::vector<int>）。

3. **引用性**：

   - 视图中的元素是对底层数据的引用，修改视图会影响原始容器。

   cpp

   ```cpp
   for (auto& v : std::views::values(m)) { v += "!"; } // 修改 map 的值
   ```

4. **空范围**：

   - 如果输入范围为空，结果视图也为空。

------

与传统方法的对比

传统循环

cpp

```cpp
std::vector<int> keys;
for (const auto& [k, v] : m) {
    keys.push_back(k);
}
```

- **缺点**：需要显式存储，立即计算。

std::views::keys

cpp

```cpp
auto keys = std::views::keys(m);
```

- **优势**：惰性、无额外存储。

------

总结

- **std::views::keys**：提取键的视图，适用于键值对范围。
- **std::views::values**：提取值的视图。 两者提供了一种声明式、惰性的方式来访问关联容器或键值对数据的一部分，与 Ranges 生态集成良好，简化代码并提升性能。如果你有具体问题或想探讨组合用法，请告诉我！