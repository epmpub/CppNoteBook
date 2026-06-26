**C++26 `<simd>` 数据并行类型（Data-Parallel Types）** 是 C++26 通过提案 **P1928R15**（以及相关提案）引入的重要数值库特性，定义在头文件 `<simd>` 中，位于命名空间 `std::simd`。

### 核心目标
提供**可移植、零开销**的类型，让开发者**显式表达数据并行性**（Data Parallelism），充分利用现代 CPU 的 SIMD 指令（如 SSE、AVX、NEON、SVE 等），而无需编写平台特定的 intrinsics 代码。

它不是简单包装 SIMD 寄存器，而是**数据并行类型**：一个对象包含多个元素，对其进行的**逐元素操作**（element-wise）会被编译器映射为 SIMD 指令或并行执行流。

### 关键概念

- **Vectorizable Types**（可向量化类型）：支持的元素类型，包括：
  - 所有标准整数和字符类型。
  - `float`、`double` 以及扩展浮点类型（如 `std::float16_t`、`std::float32_t` 等，如果实现定义）。
  - `std::complex<T>`（T 为可向量化浮点类型）。

- **Data-Parallel Type**：由 `basic_vec` 和 `basic_mask` 的特化组成。
  - **Element Type**：底层元素类型（T）。
  - **Width**（宽度）：每个对象包含的元素数量（编译期常量）。

- **Element-wise 操作**：对每个元素独立应用操作（无序），便于编译器生成 SIMD 代码。

### 主要类型

```cpp
namespace std::simd {

    template<typename T, typename Abi = /* implementation-defined */>
    class basic_vec;          // 主向量类型

    template<typename T, int N>   // 便捷别名，可指定宽度
    using vec = basic_vec<T, /* fixed-size Abi */>;

    template<typename T, typename Abi = /* ... */>
    class basic_mask;         // 掩码类型（元素为 bool）

    template<typename T, int N>
    using mask = basic_mask<T, /* ... */>;

} // std::simd
```

- `std::simd::vec<T>`（或 `std::simd::basic_vec<T>`）：最常用，类似“SIMD 向量”。
- `std::simd::mask<T>`：用于条件操作（类似掩码寄存器）。

**默认宽度**：由实现根据目标架构和 ABI 选择（例如在 AVX2 机器上 `vec<float>` 的 `size()` 可能为 8）。

### 基本用法示例

```cpp
#include <simd>
#include <vector>
#include <cmath>

void process(std::vector<float>& data) {
    using floatv = std::simd::vec<float>;   // 或 std::simd::vec<float, 8> 指定宽度

    for (auto it = data.begin(); it < data.end(); it += floatv::size()) {
        floatv v(it);           // 加载（支持未对齐/对齐加载）
        v = std::sin(v);        // 逐元素 sin（支持很多数学函数）
        v.copy_to(it);          // 写回
    }
}
```

**逐元素操作**（`+`、`-`、`*`、`/`、`min`、`max`、`clamp` 等）全部支持，像普通标量类型一样使用。

### 重要特性

- **Load / Store**：`unchecked_load`、`partial_load`、`copy_to` 等，支持对齐标志（`flag_aligned`、`flag_overaligned`）。
- **掩码（Mask）**：条件选择（`select`）、`all_of`/`any_of`/`none_of`、`reduce_count` 等。
- **Reduction（归约）**：`reduce`、`reduce_min`、`reduce_max`、`reduce_min_index` 等。
- **Cast / Permute**：`chunk`（拆分）、`cat`（拼接）、permute 操作。
- **constexpr**：几乎所有操作（除部分数学函数）都支持常量求值。
- **与 Ranges/Algorithms 集成**：未来可作为执行策略（execution policy）使用。

### 与现有技术的对比

- **优于 intrinsics**：可移植、可读性高。
- **与 Auto-Vectorization**：互补——当循环简单时，编译器自动向量化足够好；当需要显式控制（如复杂条件、混合操作）时，`std::simd` 更强大。
- **来自实验 TS**：前身是 `<experimental/simd>`（Parallelism TS 2），C++26 已正式合并。

### 注意事项

- **性能**：零开销抽象，但实际效果高度依赖编译器实现和 `-march` 等标志。早期反馈显示在某些场景下可能不如高度优化的 intrinsics 或第三方库（如 EVE）。
- **ABI Tag**：`Abi` 参数影响布局和兼容性，通常使用默认值。
- **Feature Test Macro**：
  ```cpp
  #if __cpp_lib_simd >= 2024XXL  // 具体值以实现为准
  ```

更多细节推荐参考：
- [cppreference: Data-parallel types (SIMD)](https://cppreference.com/cpp/numeric/simd)
- 提案 [P1928R15](https://wg21.link/P1928)

这是 C++26 在高性能计算领域的重要补充，让数据并行编程变得更加现代和标准化。如果你需要具体操作示例（如掩码、归约、permute）或其他细节，随时补充！