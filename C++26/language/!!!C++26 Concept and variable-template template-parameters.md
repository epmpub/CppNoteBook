### C++26 Concept and variable-template template-parameters

在 C++26 中，通过了 **[P2841R7 提案（Concept and variable-template template-parameters）](https://isocpp.org/files/papers/P2841R7.pdf)**，正式填补了模板元编程领域的一块重大空白：**允许将 Concept（概念）和 Variable Template（变量模板）作为模板的模板参数（Template Template-Parameters）进行传递**。 [1, 2] 

这项改动打破了旧标准的限制，使得元编程代码不再需要依赖臃肿的包装类，表达力大幅提升。 [3, 4] 

------

## 历史痛点：被孤立的 Concept 与变量模板

在 **C++23 及更早的标准**中，“模板的模板参数”仅限于类模板（Class Templates）或别名模板（Alias Templates）。 [1, 2] 

```cpp
template <template <typename> class T> 
struct Container; // 传统写法：只能传 class 模板或 alias 模板
```

这意味着，作为 C++14 和 C++20 核心特性的“变量模板”与“Concept”，虽然在语法上都是模板，却**无法直接作为参数传给另一个模板**。 [1, 2] 

## C++23 中的丑陋绕路法

如果你想写一个高阶模板来检查某个类型是否符合特定的 Concept（比如 `std::integral`），你不得不编写一层无意义的结构体外壳（Boilerplate Wrapper）来转接： [3] 

```cpp
// 必须手写一个包装类模板
template <typename T>
struct IsIntegralWrapper {
    static constexpr bool value = std::integral<T>;
};

// 消费端
template <template <typename> class Trait>
struct Checker {
    static_assert(Trait<int>::value);
};

Checker<IsIntegralWrapper> c; // 绕了一大圈才能传递
```

------

## C++26 的解决方案

C++26 正式将模板参数的类别扩展到了 5 种（类型、非类型、类模板、**Concept 模板**、**变量模板**）。 [5] 

## 1. Concept 作为模板参数（Concept Template-Parameter）

现在，你可以使用 `template <...> concept 名字` 语法直接声明一个接收 Concept 的模板。 [1] 

```cpp
#include <concepts>
#include <type_traits>

// 声明一个高阶 Concept：接收一个类型 T 和一个概念 C
template <typename T, template <typename> concept C>
concept Satisfies = C<T>; // 直接调用传入的概念 C

// 消费端
static_assert(Satisfies<int, std::integral>);     // 完美通过
static_assert(!Satisfies<float, std::integral>);   // 完美通过
```

## 杀手级应用：高阶组合概念（如约束 Range 元素的属性）

这在编写现代标准库或泛型容器时非常强大，比如定义一个“特定元素类型的 Range”： [6] 

```cpp
#include <ranges>

// 泛型的 range_of：第二个参数直接接收任意单参数 Concept
template <typename R, template <typename> concept Constraint>
concept range_of = std::ranges::range<R> && Constraint<std::ranges::range_reference_t<R>>;

// 优雅复用
template <typename R>
requires range_of<R, std::integral> // 直接传递 std::integral 概念！
void process_integers(R&& range);
```

## 2. 变量模板作为模板参数（Variable Template-Parameter）

同样的，你可以使用 `template <...> auto 名字` 语法来接收一个变量模板（通常用来传递编译期 traits 属性值）。 [1, 7] 

```cpp
#include <type_traits>

// 接收一个变量模板 Vt
template <typename T, template <typename> auto Vt>
struct Metric {
    static constexpr auto value = Vt<T>; 
};

// 自定义一个变量模板
template <typename T>
constexpr size_t double_sizeof = sizeof(T) * 2;

// 消费端：直接把变量模板当成参数传进去
Metric<int, double_sizeof> m; 
static_assert(m.value == 8); // sizeof(int)*2 = 8
```

------

## 完美的偏序推导与缩写语法（Subsumption & Terse Syntax）

C++26 团队在设计该提案时考虑得非常周到，这一特性与 C++20 的 **Concept 包含关系（Subsumption）** 和 **缩写函数模板语法（Terse Syntax）** 完美融合。 [6, 8] 

你可以写出像下面这样极其科幻且精简的高阶泛型算法： [8] 

```cpp
// 定义一个可以接收一类 Concept 的元包裹器
template <typename T, template <typename> concept... Constraints>
concept all_of = (Constraints<T> && ...); // 结合 C++26 折叠表达式约束推导

// 利用缩写语法，直接在参数列表中对 Concept 参数进行约束
void execute(template <typename> concept C) {
    // 动态检查或生成逻辑
}
```

## 总结

C++26 的这一改动，彻底清除了模板参数类型不统一的“历史死角”。它不仅消灭了类似 `::value` 的视觉噪音，更是让现代 C++ 库开发者在编写高阶元函数、策略模式（Policy-based design）或复杂的约束逻辑时，能够写出**如同将函数作为参数传递给高阶函数（如 Python/函数式编程）一般自然**的泛型代码。 [1, 3, 4, 9] 

如果你目前正在设计一套供团队内部使用的泛型组件或模板库，有没有哪些复杂的基类约束让你饱受老旧语法的折磨？我们可以一起尝试用 C++26 的新设计来重构、精简它！

[1] [https://www.open-std.org](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2024/p2841r2.pdf)

[2] [https://www.open-std.org](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2023/p2841r0.pdf)

[3] [https://www.sandordargo.com](https://www.sandordargo.com/blog/2025/08/20/cpp26-P2841)

[4] [https://www.sandordargo.com](https://www.sandordargo.com/blog/2025/08/20/cpp26-P2841)

[5] [https://brevzin.github.io](https://brevzin.github.io/c++/2025/08/02/ctp-reflection/)

[6] [https://www.open-std.org](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2025/p2841r6.pdf)

[7] [https://stackoverflow.com](https://stackoverflow.com/questions/79689149/why-does-c26-still-not-allow-passing-a-function-template-as-a-template-paramet)

[8] [https://isocpp.org](https://isocpp.org/files/papers/P2841R7.pdf)

[9] [https://isocpp.org](https://isocpp.org/files/papers/P2841R7.pdf)