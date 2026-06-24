#### C++26 std::atomic（及 std::atomic_ref）对浮点类型的 fetch_min / fetch_max 支持

（提案 P3008R6 + P0493 相关部分，已纳入 C++26）。这是 C++26 在原子操作方面的重要扩展，之前整数/指针类型已有 fetch_min/fetch_max，现在浮点类型（float、double、long double 及扩展浮点类型）也正式支持。1. 为什么需要专门的浮点版本？浮点数的 min/max 语义比整数复杂得多，主要难点在于：

- NaN（Not a Number） 处理：是当作“缺失数据”返回另一个值，还是当作错误传播 NaN？
- Signed Zero（-0.0 vs +0.0）：两者是否等价？-0.0 < +0.0 是否成立？
- std::min / std::max 的问题：对 NaN 是 Undefined Behavior（不适合并发原子操作）。

直接用 std::min/std::max 语义在浮点原子操作上不安全，也不符合 IEEE 754 实践和硬件指令行为。2. 接口

cpp

```cpp
namespace std {
    template<> struct atomic<floating-point> {
        // C++26 新增
        floating-point fetch_min(floating-point val,
                                 memory_order = memory_order::seq_cst) noexcept;
        floating-point fetch_max(floating-point val,
                                 memory_order = memory_order::seq_cst) noexcept;

        // 可能还有更明确的变体（fminimum_num 等）
        floating-point fetch_fminimum(...);
        floating-point fetch_fmaximum(...);
        floating-point fetch_fminimum_num(...);
        floating-point fetch_fmaximum_num(...);
    };

    // 同样适用于 std::atomic_ref<floating-point>
}
```

还有对应的自由函数：std::atomic_fetch_min、std::atomic_fetch_max 等。3. 语义（C++26 最终选择）提案经过多次讨论，最终为 fetch_min / fetch_max 选择了实用且高性能的语义（接近硬件指令和 IEEE 754 推荐）：

- Quiet NaN（qNaN）：当作“缺失数据”（Missing Data）处理 —— 如果一个操作数是 NaN，返回另一个操作数（类似 std::fmin / std::fmax）。
- Signed Zero：鼓励实现 -0.0 < +0.0（但可能作为 QoI）。
- 如果两者都是 NaN：返回 NaN。
- 性能：直接映射到硬件原子 min/max 指令（GPU、现代 CPU 都有支持），性能远优于 CAS 循环。

这些语义与 std::fmin/std::fmax 高度一致，但原子操作需要更强的保证。4. 使用示例

cpp

```cpp
#include <atomic>
#include <iostream>

int main() {
    std::atomic<double> min_val{std::numeric_limits<double>::infinity()};
    std::atomic<double> max_val{-std::numeric_limits<double>::infinity()};

    // 多线程中安全更新全局最小/最大值
    auto update = [&](double x) {
        min_val.fetch_min(x);   // 原子更新最小值
        max_val.fetch_max(x);   // 原子更新最大值
    };

    // ... 多线程调用 update ...
}
```

典型应用场景：

- 并行计算全局最小/最大值（科学计算、机器学习、渲染等）。
- 原子归约（reduction）操作。
- GPU-like 高线程数场景（提案中展示了极高线程数下的吞吐量优势）。
- 与整数版本对比

| 特性        | 整数/指针版本       | 浮点版本 (C++26)                |
| ----------- | ------------------- | ------------------------------- |
| 语义基础    | std::min / std::max | IEEE 754 风格（fmin/fmax-like） |
| NaN 处理    | 不适用              | 特殊处理（Missing Data）        |
| Signed Zero | 不适用              | 关注 -0.0 / +0.0                |
| 性能        | 优秀                | 优秀（硬件指令映射）            |

6. 特性测试宏

cpp

```cpp
#ifdef __cpp_lib_atomic_min_max
// C++26 可用
#endif
```

总结C++26 为 std::atomic<float/double/...> 增加了 fetch_min 和 fetch_max，填补了高性能并行代码中原子浮点归约的空白。它避免了 std::min 的 UB 问题，选择了符合 IEEE 754 和硬件实际的语义，同时保持了简单易用的接口。这是原子操作从“整数为主”向“现代数值计算友好”演进的重要一步，尤其适合科学计算、游戏、机器学习等领域。需要 fetch_fminimum_num 等变体细节、完整语义表格，还是与 std::execution 并行算法结合的例子吗？