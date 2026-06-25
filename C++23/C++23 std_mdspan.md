# C++23 *std::mdspan*

C++23 *std::mdspan*是连续元素序列的视图，提供这些元素上的多维数组的接口。

数组维度可以静态或动态指定。此外，*std::mdspan*支持数据布局，包括定义自定义布局的选项。



```C++
#include <mdspan>
#include <vector>
#include <numeric>
#include <ranges>
#include <print>

int main() {
    std::vector<int> arr(24);
    std::iota(arr.begin(), arr.end(), 0);

    // 3d array with runtime specified dimensions
    auto v1 = std::mdspan(arr.data(), 3, 4, 2);

    // Size of each dimension can be accessed using the extent method
    for (auto i : std::views::iota(0uz, v1.extent(0))) {
        for (auto j : std::views::iota(0uz, v1.extent(1))) {
            for (auto k : std::views::iota(0uz, v1.extent(2))) {
                // Elements can be accessed using the C++23 
                // multi-dimensional subscript operator
                // v1[i,j,k] == arr[i*(4*2)+j*2+k]
                std::print("{} ", v1[i,j,k]);
            }
            std::println("");
        }
        std::println("");
    }

    // We can define a custom type of index and custom dimensions
    using custom_extents = std::extents<uint8_t, std::dynamic_extent, 12>;
    // std::mdspan with uint8_t as the index type, runtime sized first dimension
    // and static second dimension
    auto v2 = std::mdspan<int, custom_extents>(arr.data(), arr.size()/12);

    // Because the second dimension is statically sized,
    // v2.extent(1) is a constant expression
    static_assert(v2.extent(1) == 12);
    // decltype(v2.extent(1)) == uint8_t
    static_assert(std::is_same_v<decltype(v2.extent(1)),uint8_t>);

    // Statically sized dimensions will generally lead to better codegen
    for (uint8_t i = 0; i != v2.extent(0); ++i)
        // The length of this loop is known at compile time
        for (uint8_t j = 0; j != v2.extent(1); ++j)
            std::print("{} ", v2[i,j]);
}
```

这段代码展示了 C++23 中引入的 <mdspan> 库，用于表示多维数组视图（multidimensional span）。代码通过两个示例展示了 std::mdspan 的用法，包括运行时维度、自定义索引类型和静态维度优化。以下是逐步解释。

------

代码概览

- 使用 std::mdspan 创建 3D 数组视图，访问元素。
- 定义自定义维度类型，展示静态维度带来的编译期优势。

------

关键组件

1. **头文件**

cpp

```cpp
#include <mdspan>
#include <vector>
#include <numeric>
#include <ranges>
```

- <mdspan>：提供 std::mdspan 和 std::extents。
- <vector>：提供 std::vector。
- <numeric>：提供 std::iota。
- <ranges>：提供 std::views::iota。
- **数据初始化**

cpp

```cpp
std::vector<int> arr(24);
std::iota(arr.begin(), arr.end(), 0);
```

- **arr**：
  - 创建包含 24 个整数的向量，初始化为 0 到 23。
- **运行时维度的 mdspan**

cpp

```cpp
auto v1 = std::mdspan(arr.data(), 3, 4, 2);
for (auto i : std::views::iota(0uz, v1.extent(0)))
    for (auto j : std::views::iota(0uz, v1.extent(1)))
        for (auto k : std::views::iota(0uz, v1.extent(2)))
            std::print("{} ", v1[i,j,k]);
```

- **std::mdspan(arr.data(), 3, 4, 2)**：

  - 创建 3D 视图，维度为 3 x 4 x 2（总共 24 个元素）。
  - 默认索引类型为 size_t，维度为运行时指定。

- **extent(N)**：

  - 返回第 N 维的大小（0 为 3，1 为 4，2 为 2）。

- **v1[i,j,k]**：

  - C++23 多维下标运算符，计算线性索引：i * (4 * 2) + j * 2 + k。

- **循环**：

  - 使用 std::views::iota 遍历每维，输出所有元素。

- **输出**：

  ```text
  0 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15 16 17 18 19 20 21 22 23
  ```

- **自定义维度的 mdspan**

cpp

```cpp
using custom_extents = std::extents<uint8_t, std::dynamic_extent, 12>;
auto v2 = std::mdspan<int, custom_extents>(arr.data(), arr.size()/12);
static_assert(v2.extent(1) == 12);
for (uint8_t i = 0; i != v2.extent(0); ++i)
    for (uint8_t j = 0; j != v2.extent(1); ++j)
        std::print("{} ", v2[i,j]);
```

- **std::extents**：

  - 定义维度类型：
    - uint8_t：索引类型。
    - std::dynamic_extent：第一维运行时指定。
    - 12：第二维静态指定。

- **v2**：

  - 第一维为 arr.size()/12 = 2，第二维为 12。
  - 视图表示 2 x 12 的 2D 数组。

- **extent(1)**：

  - 静态维度，编译期常量，值为 12。

- **static_assert**：

  - 验证第二维为常量。

- **循环**：

  - 第一维动态（2），第二维静态（12）。
  - 输出与 v1 相同，但布局为 2 x 12。

- **输出**：

  ```text
  0 1 2 3 4 5 6 7 8 9 10 11 12 13 14 15 16 17 18 19 20 21 22 23
  ```

------

为什么这样工作？

1. **std::mdspan**：
   - 非拥有型视图，引用底层数据，提供多维访问。
   - 支持运行时和编译时维度。
2. **extent(N)**：
   - 返回维度大小，静态维度可用于编译期优化。
3. **多维下标**：
   - v1[i,j,k] 和 v2[i,j] 映射到线性索引。
4. **静态维度**：
   - v2.extent(1) 是常量，编译器可展开内层循环。

------

优化优势

- **运行时维度（v1）**：
  - 灵活但代码生成较通用。
- **静态维度（v2）**：
  - 编译器知道 extent(1) == 12，可优化循环（如展开或向量化）。

------

使用场景

- **多维数据**：
  - 表示矩阵或张量。
- **性能优化**：
  - 静态维度提升代码生成效率。
- **通用性**：
  - 运行时维度适应动态大小。

------

总结

- v1：3D 视图（3 x 4 x 2），运行时维度。
- v2：2D 视图（2 x 12），自定义索引和静态维度。
- std::mdspan 提供灵活的多维数组抽象。
- 代码展示了其用法和优化潜力。