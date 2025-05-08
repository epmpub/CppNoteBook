# std::three_way_comparable_with

在 C++20 中，std::three_way_comparable_with 是一个概念（concept），定义在 <concepts> 头文件中，用于约束两个类型 T 和 U，确保它们之间支持三路比较运算符 <=>（spaceship operator），并且比较结果符合特定的比较类别（comparison category）。

它是对 std::three_way_comparable 的扩展，专注于**跨类型**的三路比较。

------

定义

std::three_way_comparable_with<T, U, C = std::partial_ordering> 要求：

1. 类型 T 和 U 之间支持三路比较运算符 <=>，即表达式 t <=> u（其中 t 是 T 类型，u 是 U 类型）是合法的。
2. 三路比较的结果类型必须是 C 或其派生类型（C 默认为 std::partial_ordering）。
3. 比较结果必须满足 C 所定义的比较语义（例如 std::strong_ordering, std::weak_ordering 或 std::partial_ordering）。

比较类别

- **std::strong_ordering**：严格完全有序，a == b 意味着 a 和 b 在所有上下文中等价（如 int）。
- **std::weak_ordering**：弱有序，允许等价但不完全相同的元素（如大小写无关的字符串比较）。
- **std::partial_ordering**：部分有序，允许某些元素不可比较（如浮点数的 NaN）。

------

与 std::three_way_comparable 的区别

- **std::three_way_comparable<T, C>**：约束单一类型 T 自身支持 <=>，且比较结果符合类别 C。
- **std::three_way_comparable_with<T, U, C>**：约束类型 T 和 U 之间的跨类型三路比较，确保 T 和 U 之间可以用 <=> 比较，并返回类别 C。

------

用法

std::three_way_comparable_with 通常用于模板编程中，以确保模板参数之间支持三路比较。例如：

cpp

```cpp
#include <concepts>
#include <compare>
#include <iostream>

template<typename T, typename U>
requires std::three_way_comparable_with<T, U>
void compare(T t, U u) {
    auto result = t <=> u;
    if (result < 0) std::cout << t << " < " << u << '\n';
    else if (result > 0) std::cout << t << " > " << u << '\n';
    else std::cout << t << " == " << u << '\n';
}

int main() {
    compare(5, 10);     // 合法：int 和 int 支持 strong_ordering
    compare(3.14, 2);   // 合法：double 和 int 支持 partial_ordering
}
```

在这个例子中，compare 函数要求 T 和 U 满足 std::three_way_comparable_with，即它们之间可以用 <=> 进行比较。

------

指定比较类别

可以显式要求特定的比较类别。例如，要求 strong_ordering：

cpp

```cpp
template<typename T, typename U>
requires std::three_way_comparable_with<T, U, std::strong_ordering>
void strong_compare(T t, U u) {
    auto result = t <=> u;
    if (result == std::strong_ordering::less) std::cout << t << " < " << u << '\n';
    else if (result == std::strong_ordering::greater) std::cout << t << " > " << u << '\n';
    else std::cout << t << " == " << u << '\n';
}
```

这里，T 和 U 之间的 <=> 比较必须返回 std::strong_ordering 类型。float 和 double 不满足此要求（因为它们返回 std::partial_ordering）。

------

实际应用

1. **跨类型比较**：当需要确保两个不同类型（如 int 和 double，或自定义类型）支持三路比较时，使用 std::three_way_comparable_with。
2. **模板约束**：在泛型编程中，限制模板参数以确保它们之间可以安全地进行比较。
3. **自定义类型**：为自定义类型实现 <=> 运算符，以支持跨类型比较。

示例：自定义类型

cpp

```cpp
#include <compare>
#include <iostream>

struct Point {
    int x, y;
    auto operator<=>(const Point& other) const = default; // 自动生成 strong_ordering
};

struct Vec {
    int x, y;
    auto operator<=>(const Point& other) const { // 跨类型比较
        return (x + y) <=> (other.x + other.y); // 自定义比较逻辑
    }
};

template<typename T, typename U>
requires std::three_way_comparable_with<T, U>
void compare(T t, U u) {
    auto result = t <=> u;
    if (result < 0) std::cout << "T < U\n";
    else if (result > 0) std::cout << "T > U\n";
    else std::cout << "T == U\n";
}

int main() {
    Point p{1, 2};
    Vec v{2, 1};
    compare(p, v); // 合法：Point 和 Vec 支持三路比较
}
```

在这个例子中，Point 和 Vec 定义了跨类型的 <=> 运算符，满足 std::three_way_comparable_with<Point, Vec>。

------

与 std::totally_ordered_with 的关系

- **std::totally_ordered_with<T, U>**：要求 T 和 U 之间支持传统比较运算符（<, >, ==, 等），并满足完全有序。
- **std::three_way_comparable_with<T, U>**：要求 T 和 U 之间支持 <=> 运算符，并返回指定的比较类别。
- 如果 T 和 U 之间的 <=> 返回 std::strong_ordering 或 std::weak_ordering，通常也能满足 std::totally_ordered_with，因为 <=> 可以推导传统比较运算符。

------

注意事项

- 如果类型之间的 <=> 运算符未定义，或者比较结果不符合指定的类别 C，则不满足 std::three_way_comparable_with。
- 浮点数（如 float, double）的比较返回 std::partial_ordering，因此在要求 std::strong_ordering 的场景中可能不适用。
- 自定义类型需要显式定义 <=> 运算符来支持跨类型比较。

------

总结

- **std::three_way_comparable_with<T, U, C>** 约束 T 和 U 之间支持三路比较运算符 <=>，并要求比较结果符合指定的比较类别 C。
- 它适用于需要跨类型三路比较的场景，特别是在泛型编程中。
- 通过结合 <=> 运算符和比较类别，C++20 提供了更现代和类型安全的比较机制。

如果你有更具体的问题或需要进一步的代码示例，请告诉我！