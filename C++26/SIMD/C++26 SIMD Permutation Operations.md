C++26 SIMD Permutation Operations

（提案 P2664R10/R11，已纳入 C++26）是 std::simd（数据并行类型库）的重要扩展，填补了之前在跨通道（cross-lane）重排数据方面的重大缺失。

1. 为什么需要 Permutation？SIMD 编程中，大量性能关键操作（如转置、交织（interleave）、去交织、strided 访问、LUT 查找、格式转换等）都依赖元素重排（permutation / shuffle）。硬件（如 AVX、NEON、SVE）有丰富指令支持，但早期 std::simd（Parallelism TS）对此支持薄弱。P2664 提供了可移植、高效的抽象，让代码能直接映射到硬件 permutation 指令。
2. 主要接口（位于 <simd>）

cpp

```cpp
namespace std::simd {

    // 特殊常量（用于 static permute）
    inline constexpr /* unspecified */ zero_element{};   // 置零
    inline constexpr /* unspecified */ uninit_element{}; // 未初始化（可能 UB）

    // 1. 静态/动态 permute（核心）
    template</* ... */>
    constexpr auto permute(const basic_vec<T, Abi>& v, IdxMap&& idxmap);  // 或 dynamic index

    // 2. 基于 mask 的 compress / expand（压缩/扩展）
    template</* ... */>
    constexpr auto compress(const basic_vec<T, Abi>& v, const basic_mask<...>& m, 
                            /* fill value */ = zero_element);

    template</* ... */>
    constexpr auto expand(const basic_vec<T, Abi>& v, const basic_mask<...>& m);

    // 3. 内存级 gather / scatter（带可选 mask）
    template</* ... */>
    auto unchecked_gather_from(/* range or iterator */, flags...);

    template</* ... */>
    auto partial_gather_from(/* range */, mask, flags...);

    // scatter 同理
}
```

特性测试宏：__cpp_lib_simd_permutations（202506L）3. 核心用法示例

cpp

```cpp
#include <simd>
namespace simd = std::simd;

using vfloat = simd::vec<float>;   // 或 fixed_size_vec<float, 8> 等

vfloat x = /* ... */;

// 示例1：静态 permute（编译期索引生成函数）
auto dup_even = simd::permute(x, [](size_t i) { return (i / 2) * 2; });  
// 效果： [a0,a0, a1,a1, a2,a2, ...]

// 示例2：动态 permute（用另一个 simd 作为索引）
simd::vec<int> indices = /* ... */;
auto y = simd::permute(x, indices);   // x[indices[i]]

// 示例3：compress / expand（常见于过滤、压缩存储）
simd::mask<4> m = /* ... some condition */;
auto compressed = simd::compress(x, m);           // 只保留 true 位置的元素
auto expanded   = simd::expand(compressed, m);    // 放回原位置，其余可填充
```

4. 四类主要 Permutation

| 类型            | 机制                         | 典型用途                    | 性能映射              |
| --------------- | ---------------------------- | --------------------------- | --------------------- |
| Static Permute  | 编译期 lambda 生成索引       | DupEven/Odd、Rotate、Stride | 优秀（常量折叠）      |
| Dynamic Permute | 用另一个 simd 作索引         | 任意重排、LUT               | vpermd / vperm 等     |
| Compress/Expand | 根据 mask 压缩/扩展元素      | 过滤、稀疏数据处理          | vpcompress / vpexpand |
| Gather/Scatter  | 从/到内存（带 mask + flags） | 非连续访问、转置、AOSSOA    | vgathter / vscatter   |

5. 设计亮点

- 零开销抽象：好的实现能直接生成单条硬件指令（如 vpermilps、vpshufb）。
- 支持 basic_simd_mask：掩码也可以 permute。
- 灵活输出大小：permute 可改变向量长度（resize_t）。
- 安全 vs 性能：unchecked_ 版本（UB on out-of-range）、partial_ 版本（mask 控制）。
- 与其它 C++26 特性结合：可与 std::execution::simd 执行策略、Ranges 等配合。
- 与之前方式对比

- 以前：只能用 chunk + cat（编译期，受限），或回退 intrinsics。
- 现在：可移植、高表达力、编译器友好。

注意：std::simd 整体仍在快速发展，部分实现（如 GCC）对 permutation 的支持可能还处于部分阶段。总结C++26 的 SIMD Permutation 操作让 std::simd 从“基础向量运算”升级为实用生产级数据并行工具。它是跨平台 SIMD 编程中最缺失的一环，现在终于补齐了，使得转置、格式转换、稀疏处理等常见模式能以可读、可移植的方式高效实现。这是 C++26 std::simd（P1928 + P2664 等）中最有实用价值的部分之一，尤其适合高性能计算、图像/音视频处理、科学模拟等领域。需要具体某个 permutation（如 transpose、interleave）的完整代码示例吗？