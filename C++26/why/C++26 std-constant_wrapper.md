C++26 std::constant_wrapper 解释（提案 P2781R9，已被纳入 C++26）。这是 C++26 中一个非常实用的编译期常量包装工具，主要目的是优雅、高效地将任意编译期常量作为类型传递，大幅改善 std::integral_constant 的笨重语法，同时支持更多类型（不仅仅是整数）。1. 核心动机

- std::integral_constant<int, 42> 写法太长、太啰嗦。
- 函数参数在进入函数体后会丢失 constexpr 属性，即使它是字面量或 constexpr 变量。
- 希望能方便地将任意 structural 类型（支持 NTTP 的类型）的常量值包装成类型，便于模板元编程、策略设计、编译期计算等。
- 主要接口

cpp

```cpp
namespace std {

    template <auto X, /* exposition-only adl helper */>
    struct constant_wrapper { ... };

    // 便捷变量模板（强烈推荐使用）
    template <auto X>
    inline constexpr constant_wrapper<X> cw{};
}
```

- cw<value>：最常用形式，产生一个 constant_wrapper 对象。
- 支持几乎所有structural 类型的常量（整数、浮点、数组、字符串字面量包装类型、用户定义的 structural struct 等）。
- 基本用法示例

cpp

```cpp
#include <constant_wrapper>  // 或通过 <utility> 等包含

constexpr int foo = 42;
constexpr double pi = 3.14;

template <typename T>
void process(std::constant_wrapper auto c) {
    using Val = decltype(c)::value_type;
    constexpr auto val = c.value;        // 始终是 constexpr
    // 或 static_cast<Val>(c)
}

int main() {
    process(std::cw<42>);           // 整数
    process(std::cw<3.14159>);      // 浮点
    process(std::cw<foo>);          // constexpr 变量
    process(std::cw<"hello">);      // 需要包装类型（见下文）

    // 运算符支持（非常强大）
    static_assert((std::cw<10> + std::cw<20>).value == 30);
    static_assert((std::cw<5> * std::cw<6>).value == 30);
}
```

4. 关键特性

- 值访问：
  - c.value（static constexpr）
  - static_cast<value_type>(c) 或 value_type{c}（隐式转换）
- 丰富运算符重载：几乎所有可重载运算符（+、-、*、[]、() 等），让包装后的常量可以像普通值一样参与编译期运算。
- ADL 支持：通过特殊的模板参数设计，尽量保留底层类型的 ADL（例如 operator<<）。
- 与 std::integral_constant：可以互转，逐步替代旧用法。
- 与 std::function_ref：C++26 中 function_ref 支持用 constant_wrapper 传入编译期常量回调。
- 字符串支持（注意事项）直接 std::cw<"hello"> 不直接支持（语言限制）。需要一个 structural 包装类型：

cpp

```cpp
template<size_t N>
struct strlit {
    char value[N]{};
    // 构造函数、比较、operator<< 等...
};

auto s = std::cw<strlit("hello")>;
```

（后续有 DR 讨论过字符串支持的简化，但最终版本有所调整。）6. 典型应用场景

- 替换 std::integral_constant（submdspan、strided_index_range 等）。
- 编译期 DSL（Domain Specific Language）。
- 策略模板（Policy-based design）。
- 常量参数传递（结合 consteval 构造函数更强大）。
- 与 Reflection（C++26）结合使用。
- 特性测试宏

cpp

```cpp
#ifdef __cpp_lib_constant_wrapper
// C++26 可用
#endif
```

总结std::constant_wrapper（配合 std::cw<...>）是 C++26 编译期编程体验的重大升级。它让“把常量当作类型传递”这件事变得简洁、自然且强大，远超传统的 integral_constant，同时为更多高级元编程和 DSL 打开大门。它是 C++26 “让编译期更像运行时一样好用”哲学的典型代表。需要 constant_wrapper 与运算符、字符串包装、function_ref 或具体库结合的更多示例吗？