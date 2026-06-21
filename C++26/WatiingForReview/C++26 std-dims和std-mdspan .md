std::dims 

是 C++26 为 std::mdspan（以及其底层 std::extents）新增的一个便捷别名模板（convenience alias template），极大简化了**所有维度均为动态（runtime）**的情况下的写法。

背景std::mdspan 在 C++23 中引入时，使用 std::extents 来描述维度（extents）：

cpp

```cpp
// C++23 写法
std::mdspan<int, std::extents<size_t, std::dynamic_extent, std::dynamic_extent>> m(data, rows, cols);

// 或者使用已有的 dextents（dynamic extents）
std::mdspan<int, std::dextents<size_t, 2>> m(data, rows, cols);
```

std::dextents<IndexType, Rank> 已经很方便，但 C++26 进一步引入了 std::dims，让代码更简洁、直观。std::dims 的定义与用法（C++26）

cpp

```cpp
// 在 <mdspan> 头文件中
template <std::size_t Rank, class IndexType = size_t>
using dims = extents<IndexType, /* Rank 个 dynamic_extent */>;
```

示例对比：

cpp

```cpp
#include <mdspan>
#include <vector>
#include <cstddef>

int main() {
    std::vector<int> vec(100 * 50);

    // C++23（较繁琐）
    std::mdspan<int, std::dextents<std::size_t, 2>> m1(vec.data(), 100, 50);

    // C++26：使用 std::dims（推荐）
    std::mdspan<int, std::dims<2>> m2(vec.data(), 100, 50);

    // 3D 示例
    std::mdspan<float, std::dims<3>> m3(data_ptr, 10, 20, 30);

    m2[5, 10] = 42;        // 多维下标访问
}
```

优势

- 极简：只需写维度数量（Rank），无需写 dextents 或一堆 dynamic_extent。

- 可读性更好：std::dims<2> 直接表达“2 个动态维度”。

- IndexType 默认：默认为 size_t，大多数场景够用。

- 支持混合静态/动态维度：如果你需要部分静态维度，仍需使用 std::extents：

  cpp

  ```cpp
  // 2D：第1维静态大小为 100，第2维动态
  std::mdspan<int, std::extents<size_t, 100, std::dynamic_extent>> fixed_row(data, cols);
  ```

与其他相关特性的关系（C++26）

- std::dims<Rank> —— 全动态维度（最常用）。
- std::dextents<IndexType, Rank> —— C++23 已有的等价物（dims 是其简化版）。
- std::extents<IndexType, ...> —— 最灵活，可混用静态和 std::dynamic_extent。
- submdspan（C++26）—— 支持对 mdspan 进行切片，常与 std::dims 配合使用。
- layout_left_padded / layout_right_padded 等新布局策略。

适用场景

- 科学计算、数值模拟、机器学习（矩阵、张量）。
- 需要在运行时确定维度的多维数组视图。
- 与 C++26 线性代数（std::linalg）配合使用时特别方便。

总结：std::dims 是 C++26 对 std::mdspan 的语法糖，让最常见的“全动态维度”场景写法更加简洁现代。它不改变 mdspan 的语义，只是显著提升了开发者体验。 更多细节可参考 cppreference：std::mdspan 和 std::dims。需要我给出更多实际示例（如矩阵运算、子视图切片）吗？