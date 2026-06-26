#### C++26 padded mdspan layouts 

这是 C++26 对 std::mdspan（C++23 引入的多维数组视图）的重要扩展，提案为 P2642R6。为什么需要 Padded Layouts？std::mdspan 在 C++23 中已有三种布局：

- std::layout_right（行主序，默认，C/C++ 风格）
- std::layout_left（列主序，Fortran 风格）
- std::layout_stride（任意步长，通用但开销稍大）

在实际高性能计算（HPC）、线性代数、SIMD 优化、缓存对齐等场景中，经常需要“带 padding（填充）的固定步长”，例如：

- 矩阵每行/每列分配比实际尺寸更大的内存（对齐到 64 字节、缓存行等）
- 与 BLAS（Basic Linear Algebra Subprograms）等 C/Fortran 库互操作
- 避免 false sharing 或启用向量化

之前只能用 layout_stride 实现，但它不够高效且表达力差。C++26 新增的两个布局

| 布局名称                 | 内存顺序              | Padding 位置                   | 主要用途                               |
| ------------------------ | --------------------- | ------------------------------ | -------------------------------------- |
| std::layout_left_padded  | 列主序 (Column-major) | 在最左侧维度（行）添加 padding | Fortran 风格 + 对齐，BLAS Column-major |
| std::layout_right_padded | 行主序 (Row-major)    | 在最右侧维度（列）添加 padding | C++ 风格 + 对齐，BLAS Row-major        |

两者都允许padding stride ≥ 对应维度的大小，从而实现对齐。使用示例

cpp

```cpp
#include <mdspan>
#include <vector>
#include <iostream>

int main() {
    // 分配带 padding 的内存（例如每行对齐到 8 的倍数）
    constexpr size_t padding = 8;
    std::vector<double> data(100 * padding);   // 100 行，每行至少 padding 个元素

    // 创建带 padding 的 mdspan（行主序）
    std::mdspan<double, std::dextents<size_t, 2>, std::layout_right_padded<padding>>
        mat(data.data(), 100, 60);   // 逻辑尺寸 100×60，实际每行 stride = padding

    std::cout << "Extent: " << mat.extent(0) << " x " << mat.extent(1) << '\n';
    std::cout << "Stride(0): " << mat.stride(0) << '\n';  // 通常是 padding
    std::cout << "Stride(1): " << mat.stride(1) << '\n';  // 1

    mat[5, 10] = 3.14;   // 正常访问
}
```

- layout_right_padded<Pad>：Pad 是右侧维度的最小 stride（通常是列数方向）
- layout_left_padded<Pad>：Pad 是左侧维度的最小 stride

如果你不指定模板参数，也可以动态指定 padding。主要优势

- 零开销抽象：编译期已知 padding 时，访问性能与手写代码相同。
- 更好的互操作：直接支持 BLAS 等库常用的 padded 布局。
- 与 std::linalg（C++26 线性代数库） 深度集成。
- 支持 submdspan 等其他 mdspan 新特性。

一句话总结：
C++26 通过 layout_left_padded 和 layout_right_padded 让 std::mdspan 能原生、高效地表达带对齐填充的矩阵/张量，极大提升了在科学计算、机器学习、性能敏感代码中的实用性，是 mdspan 在 C++26 中最重要的一项增强。需要我给你更多示例（比如与 submdspan 结合、动态 padding、或性能对比）吗？