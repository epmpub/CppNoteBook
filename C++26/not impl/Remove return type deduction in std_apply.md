C++26 中 std::apply 的重大变更（提案 P1317R2 "Remove return type deduction in std::apply")。这是对 C++17 引入的 std::apply 的重要改进，主要解决了返回类型推导带来的问题，并新增了三个配套工具。1. 为什么要做这个变更？旧问题（C++17 ~ C++23）：

- std::apply 使用 decltype(auto) 返回类型推导。
- 这导致在某些场景下返回类型难以预测，尤其涉及 void、引用、prvalue 等复杂情况。
- 与 std::invoke、std::invoke_r 等不一致。
- 无法方便地提前查询 std::apply(f, t) 的返回类型（元编程不友好）。
- SFINAE / Concepts 友好度差。

C++26 解决方案：移除 decltype(auto) 推导，改为显式返回类型 + 新 trait。2. 新增的类型特性（Traits）

cpp

```cpp
#include <tuple>   // 或 <utility>

namespace std {

    // 1. 返回类型查询
    template<class F, class Tuple>
    struct apply_result;                     // primary template

    template<class F, class Tuple>
    using apply_result_t = typename apply_result<F, Tuple>::type;

    // 2. 可调用性检查
    template<class F, class Tuple>
    inline constexpr bool is_applicable_v = /* ... */;

    template<class F, class Tuple>
    struct is_applicable;                    // 可用于 requires / SFINAE

    // 3. 不抛异常检查
    template<class F, class Tuple>
    inline constexpr bool is_nothrow_applicable_v = /* ... */;

    template<class F, class Tuple>
    struct is_nothrow_applicable;
}
```

3. std::apply 签名变更（C++26）

cpp

```cpp
// C++23 及之前（简化版）
template<class F, class Tuple>
constexpr decltype(auto) apply(F&& f, Tuple&& t);

// C++26
template<class F, tuple-like Tuple>
constexpr apply_result_t<F, Tuple>     // 显式使用新 trait
    apply(F&& f, Tuple&& t)
        noexcept(is_nothrow_applicable_v<F, Tuple>);
```

4. 使用示例

cpp

```cpp
#include <tuple>
#include <utility>
#include <concepts>

int add(int a, int b) { return a + b; }
void do_something() { }

int main() {
    auto t = std::tuple{10, 20};

    // 1. 直接使用 apply
    auto result = std::apply(add, t);           // int

    // 2. 查询返回类型（元编程友好）
    using Ret = std::apply_result_t<decltype(add), decltype(t)>;  // int

    static_assert(std::same_as<Ret, int>);

    // 3. Concepts / requires
    if constexpr (std::is_applicable_v<decltype(add), decltype(t)>) {
        // ...
    }

    // 4. noexcept 检查
    static_assert(std::is_nothrow_applicable_v<decltype(add), decltype(t)>);
}
```

与 std::invoke_result 风格完全一致，提升了接口统一性。5. 特性测试宏

cpp

```cpp
#ifdef __cpp_lib_apply
// C++26 更新后的值（通常为 2025xxL）
#endif
```

总结C++26 对 std::apply 的改进主要目标是：

- 消除返回类型推导的不确定性。
- 提供标准化的元编程支持（apply_result、is_applicable、is_nothrow_applicable）。
- 与 std::invoke / std::invoke_r 等工具保持一致的设计风格。

这对泛型库作者、元编程和高阶函数使用场景非常友好，是一个“干净、现代”的小改进。需要 apply_result 与 std::tuple、std::invoke 的对比示例，或在泛型代码中的实际应用场景吗？