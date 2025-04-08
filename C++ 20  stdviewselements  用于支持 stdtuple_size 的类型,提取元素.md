# C++ 20  std::views::elements  用于支持 std::tuple_size 的类型,提取元素

```C++
#include <ranges>
#include <vector>
#include <array>
#include <print>

int main() {
    std::vector<std::pair<int,double>> data{{1,2.7}, {3, 4.2}, {-1, 3.3}};

    for (auto v : data | std::views::elements<0>) {
        std::print("{} ", v);
    }
    std::println("");

    for (auto v : data | std::views::elements<1>) {
        std::print("{} ", v);
    }
    std::println("");

    std::array<std::array<int,3>,3> grid{1,2,3,4,5,6,7,8,9};
    for (auto v : grid | std::views::elements<2>) {
        std::print("{} ", v);
    }
    std::println("");
}
```

这段代码展示了 C++20 中 <ranges> 库提供的 std::views::elements 视图，用于从容器（如 std::vector 或 std::array）的元素中提取指定索引的成员。代码通过示例展示了如何从 std::pair 和嵌套数组中提取特定元素并输出。以下是逐步解释。

------

代码概览

- 使用 std::views::elements 从 std::vector<<std::pair>> 中提取第一和第二元素。
- 使用 std::views::elements 从 std::array<std::array<int, 3>, 3> 中提取第三个元素。

------

关键组件

1. **头文件**

cpp

```cpp
#include <ranges>
#include <vector>
#include <array>
#include <print>
```

- <ranges>：提供 std::views::elements。
- <vector>：提供 std::vector。
- <array>：提供 std::array。
- <print>：C++23 的 std::print 和 std::println 用于输出。
- **std::views::elements 与 std::pair**

cpp

```cpp
std::vector<std::pair<int,double>> data{{1,2.7}, {3, 4.2}, {-1, 3.3}};
for (auto v : data | std::views::elements<0>) {
    std::print("{} ", v);
}
std::println("");
for (auto v : data | std::views::elements<1>) {
    std::print("{} ", v);
}
std::println("");
```

- **data**：

  - 包含 3 个 std::pair<int, double>：{{1, 2.7}, {3, 4.2}, {-1, 3.3}}。

- **std::views::elements<N>**：

  - 从容器元素中提取第 N 个成员（索引从 0 开始）。
  - 对于 std::pair，0 表示 first，1 表示 second。

- **调用 1**：

  - data | std::views::elements<0>：
    - 提取每个 pair 的 first：{1, 3, -1}。

- **调用 2**：

  - data | std::views::elements<1>：
    - 提取每个 pair 的 second：{2.7, 4.2, 3.3}。

- **输出**：

  ```text
  1 3 -1
  2.7 4.2 3.3
  ```

- **std::views::elements 与嵌套数组**

cpp

```cpp
std::array<std::array<int,3>,3> grid{1,2,3,4,5,6,7,8,9};
for (auto v : grid | std::views::elements<2>) {
    std::print("{} ", v);
}
std::println("");
```

- **grid**：

  - 3x3 数组，初始化为 {{1, 2, 3}, {4, 5, 6}, {7, 8, 9}}。
  - 注意：初始化语法扁平化填充。

- **std::views::elements<2>**：

  - 从每个内部 std::array<int, 3> 提取第 2 个元素（索引从 0 开始）。
  - 提取第 3 列：{3, 6, 9}。

- **输出**：

  ```text
  3 6 9
  ```

------

为什么这样工作？

1. **std::views::elements**：
   - 适用于支持 std::tuple_size 的类型（如 std::pair、std::array）。
   - 返回一个视图，仅包含指定索引的元素。
2. **管道操作符 |**：
   - 将容器与视图组合，生成新的范围。
3. **范围 for 循环**：
   - 遍历视图，访问提取的元素。

------

输出

```text
1 3 -1
2.7 4.2 3.3
3 6 9
```

------

使用场景

- **数据提取**：
  - 从键值对或多维数组中提取特定字段。
- **视图操作**：
  - 非侵入式处理容器元素。
- **简化代码**：
  - 替代手动循环和索引访问。

------

总结

- std::views::elements<0> 从 std::pair 提取 first：{1, 3, -1}。
- std::views::elements<1> 提取 second：{2.7, 4.2, 3.3}。
- std::views::elements<2> 从嵌套数组提取第 3 列：{3, 6, 9}。
- 代码展示了 Ranges 视图的强大功能和简洁性。