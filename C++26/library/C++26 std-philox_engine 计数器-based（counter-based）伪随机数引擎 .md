std::philox_engine 是 C++26 在 <random> 头文件中引入的计数器-based（counter-based）伪随机数引擎（Uniform Random Bit Generator）。它是 C++11 之后第一个新增的标准随机数引擎，专为高性能并行计算（尤其是 Monte Carlo 模拟、GPU/向量计算）设计。 

en.cppreference.com

核心特点（Counter-Based RNG）

- 不依赖传统状态演化：传统引擎（如 mt19937）通过状态转换生成下一个数；而 Philox 使用一个大**计数器（counter）**作为输入，通过固定轮数的加密-like 混淆（rounds）直接计算输出。
- 状态很小：内部主要是一个多字计数器（n × w bits），适合大规模并行。
- 极长的周期：Philox4x32 的周期至少为 2¹³⁰。
- 易并行化：每个线程/工作项只需设置不同的counter 或 key，即可得到独立随机流，无需担心状态竞争。
- 可向量化和 GPU 友好：算法高度可并行，常用于 PyTorch、CUDA 等框架。

模板参数

cpp

```cpp
template <
    class UIntType,          // 输出类型（如 uint32_t / uint64_t）
    std::size_t w,           // 字宽（bits），通常 32 或 64
    std::size_t n,           // 字数（2 或 4）
    std::size_t r,           // 轮数（rounds）
    UIntType ... consts      // 乘数和轮常量
>
class philox_engine;
```

- n=4 最常用（Philox4x*）。
- r=10 是常用安全轮数。

预定义别名（最常用）

cpp

```cpp
using philox4x32 = std::philox_engine<std::uint_fast32_t, 32, 4, 10,
    0xCD9E8D57, 0x9E3779B9, 0xD2511F53, 0xBB67AE85>;

using philox4x64 = std::philox_engine<std::uint_fast64_t, 64, 4, 10,
    0xCA5A826395121157, 0x9E3779B97F4A7C15,
    0xD2E7470EE14C6C93, 0xBB67AE8584CAA73B>;
```

与传统引擎对比

| 引擎               | 类型                           | 状态大小 | 周期   | 并行友好度 | 典型用途                                   |
| ------------------ | ------------------------------ | -------- | ------ | ---------- | ------------------------------------------ |
| std::mt19937       | 状态演化                       | ~2.5KB   | 2¹⁹⁹³⁷ | 一般       | 通用高质量随机                             |
| std::minstd_rand   | 线性同余                       | 很小     | 较短   | 一般       | 简单快速                                   |
| std::philox_engine | Counter-based 基于计数器的机制 | 很小     | 极大   | 极高       | 并行、GPU、Monte Carlo 并行、GPU、蒙特卡洛 |

基本用法示例

cpp

```cpp
#include <random>
#include <iostream>

int main() {
    std::philox4x32 rng;           // 默认构造（counter=0）
    // 或 std::philox4x32 rng{seed};

    // 设置特定计数器（关键特性！支持并行）
    rng.set_counter(123456ULL);    // C++26 新增成员

    for (int i = 0; i < 10; ++i) {
        std::cout << rng() << '\n';   // 生成 uint_fast32_t 随机数
    }

    // 可用于分布
    std::uniform_real_distribution<double> dist(0.0, 1.0);
    double r = dist(rng);
}
```

重要成员（C++26）

- operator() —— 生成下一个随机数
- seed() —— 用种子重置
- set_counter() —— 直接设置计数器（支持独立流）
- discard(z) —— 跳过 z 个随机数
- min() / max() —— 最小/最大可能值
- 支持 ==、<<、>>（序列化）

适用场景

- 大规模并行 Monte Carlo 模拟（金融、物理、机器学习）。
- GPU / 多线程 / SIMD 环境。
- 需要可重复、可跳跃（jumpable）、统计性质好的随机数。
- 希望状态小、速度快、易向量化的场景。

总结：std::philox_engine 是 C++26 为现代高性能计算量身定制的随机数引擎。它在并行友好性和小状态上远超传统引擎，是需要大量独立随机流的首选。 

cppreference.com

更多细节可参考 cppreference：std::philox_engine。