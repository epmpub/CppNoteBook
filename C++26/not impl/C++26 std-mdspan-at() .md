C++26 std::mdspan::at() 

是对 std::mdspan（C++23 引入）的重要补充，提供带边界检查的多维索引访问。1. 背景与动机

- std::mdspan 的主要访问方式是 operator[]（多维 md[i, j, k]），不进行边界检查（追求零开销）。
- 类似于 std::span::at()（C++26 也新增）、std::vector::at()、std::array::at() 等，mdspan 也需要一个安全的访问接口，尤其在调试、测试和安全性要求高的场景中。
- 提案：与 span::at() 类似（P2821），mdspan::at() 在 C++26 中被采纳。
- 接口

cpp

```cpp
template <class T, class Extents, class LayoutPolicy, class AccessorPolicy>
class mdspan {
public:
    // C++26 新增
    constexpr reference at(size_type i0, size_type i1, ..., size_type iR-1);  // 精确 rank 个参数
    constexpr reference at(std::array<size_type, rank()> indices);           // 或使用 array
    // 可能还有其他重载（如 spans 等，视最终标准而定）
};
```

- 参数：必须提供与 rank() 完全匹配的索引（或等价的 std::array）。
- 返回值：reference（通常是 AccessorPolicy::reference）。
- 异常：如果任意索引越界，抛出 std::out_of_range。
- 复杂度：边界检查 + operator[] 的开销（通常为常数时间）。
- 使用示例

cpp

```cpp
#include <mdspan>
#include <vector>
#include <iostream>
#include <stdexcept>

int main() {
    std::vector<int> data(24);
    std::mdspan<int, std::dextents<size_t, 2, 3, 4>> md(data.data(), 2, 3, 4);

    // 安全访问（C++26）
    try {
        md.at(1, 2, 3) = 999;           // 正常
        std::cout << md.at(1, 2, 3) << '\n';

        std::cout << md.at(2, 0, 0) << '\n';  // 抛出 std::out_of_range（第一维只有 0..1）
    } catch (const std::out_of_range& e) {
        std::cout << "Error: " << e.what() << '\n';
    }

    // 与 unsafe operator[] 对比
    md[1, 2, 3] = 42;   // 不检查，潜在 UB 如果越界
}
```

4. 与 operator[] 对比

| 特性         | md[i, j, k] (operator[]) | md.at(i, j, k) (C++26)   |
| ------------ | ------------------------ | ------------------------ |
| 边界检查     | 无（零开销）             | 有（抛 out_of_range）    |
| 性能         | 最高                     | 稍慢（检查开销）         |
| 适用场景     | 性能关键、内层循环       | 调试、边界安全、用户输入 |
| 异常         | 无                       | 可能抛出                 |
| 表达式友好度 | 极佳（md[1,2,3]）        | 类似                     |

5. 设计细节

- 一致性：遵循 C++ 容器/视图的惯例（[] unsafe，at() safe）。
- 多维支持：支持与 operator[] 相同的索引风格（C++23 多维下标）。
- Accessor：at() 最终通过 accessor 的 access() 实现，边界检查在 mdspan 层完成。
- 特性测试宏：包含在 __cpp_lib_mdspan 的更新值中（C++26）。

总结std::mdspan::at() 是 C++26 中一个实用性改进，让 mdspan 在需要安全访问时更加完整。它与 submdspan（C++26）、padded layouts、aligned_accessor 等一起，使 mdspan 成为真正生产就绪的多维数组视图。推荐实践：

- 性能关键路径用 operator[]。
- 调试、初始化、用户控制的索引用 at()。
- 结合 std::span::at() 使用，形成一致的安全访问风格。

需要 at() 与 submdspan 结合的例子、或性能对比细节吗？