# std::totally_ordered_with 和 three_way_comparable 

在 C++20 中，std::totally_ordered_with 和 three_way_comparable 是与比较操作相关的新概念，主要用于支持新的比较运算符 <=>（三路比较运算符，spaceship operator）以及提供类型安全和一致的比较语义。以下是对这两个概念的解释：

------

1. **std::totally_ordered_with**

std::totally_ordered_with 是一个概念（concept），定义在 <concepts> 头文件中，用于约束两个类型 T 和 U，确保它们之间可以通过比较运算符（<, >, <=, >=, ==, !=）进行完全有序的比较。

定义

std::totally_ordered_with<T, U> 要求：

- 类型 T 和 U 之间支持所有比较运算符（==, !=, <, >, <=, >=）。
- 这些比较运算符的结果必须满足完全有序（total ordering）的数学性质：
  - **反对称性**：若 a < b，则 !(b < a)。
  - **传递性**：若 a < b 且 b < c，则 a < c。
  - **完全性**：对于任意 a 和 b，要么 a < b，要么 b < a，要么 a == b。
- 比较操作必须是类型安全的，且结果是可预测的。

用法

std::totally_ordered_with 通常用于模板编程中，以确保模板参数支持完全有序的比较。例如：

cpp

```cpp
#include <concepts>
#include <iostream>

template<typename T, typename U>
requires std::totally_ordered_with<T, U>
void compare(T t, U u) {
    if (t < u) std::cout << t << " < " << u << '\n';
    else if (t > u) std::cout << t << " > " << u << '\n';
    else std::cout << t << " == " << u << '\n';
}

int main() {
    compare(5, 10);   // 合法：int 和 int 满足 totally_ordered_with
    compare(3.14, 2); // 合法：double 和 int 满足 totally_ordered_with
}
```

在这个例子中，compare 函数要求 T 和 U 满足 std::totally_ordered_with，否则编译会失败。

注意

- 如果类型之间不支持比较（例如自定义类型未定义比较运算符），或者比较不满足完全有序（例如浮点数的 NaN），则不满足 std::totally_ordered_with。
- std::totally_ordered 是 std::totally_ordered_with<T, T> 的特例，用于约束单一类型自身是否完全有序。

------

2. **three_way_comparable**

three_way_comparable 是一个概念，也定义在 <concepts> 头文件中，用于约束类型支持三路比较运算符 <=>，并确保其比较结果符合特定的语义。

定义

three_way_comparable<T, C = std::partial_ordering> 要求：

- 类型 T 支持三路比较运算符 <=>。
- 三路比较的结果类型是 C（默认是 std::partial_ordering）。
- 如果指定了 C（例如 std::strong_ordering 或 std::weak_ordering），则比较结果必须符合 C 所要求的比较语义。

三路比较的结果类型

C++20 引入了三种比较类别（comparison categories），用于描述三路比较的结果：

1. **std::strong_ordering**：
   - 表示完全严格的有序关系（如整数比较）。
   - 满足：a == b 意味着 a 和 b 在所有上下文中可互换。
   - 例子：int, std::string。
2. **std::weak_ordering**：
   - 表示弱有序关系，允许某些等价但不完全相同的元素（如大小写无关的字符串比较）。
   - 例子：自定义类型可能定义弱序。
3. **std::partial_ordering**：
   - 表示部分有序，允许某些元素不可比较（如浮点数的 NaN）。
   - 例子：float, double。

用法

three_way_comparable 常用于模板编程中，以确保类型支持 <=> 运算符，并返回正确的比较类别。例如：

cpp

```cpp
#include <concepts>
#include <compare>
#include <iostream>

template<typename T>
requires std::three_way_comparable<T>
void compare(T a, T b) {
    auto result = a <=> b;
    if (result < 0) std::cout << a << " < " << b << '\n';
    else if (result > 0) std::cout << a << " > " << b << '\n';
    else std::cout << a << " == " << b << '\n';
}

int main() {
    compare(5, 10);     // 合法：int 支持 strong_ordering
    compare(3.14, 2.0); // 合法：double 支持 partial_ordering
}
```

指定比较类别

可以显式要求特定的比较类别。例如，要求 strong_ordering：

cpp

```cpp
template<typename T>
requires std::three_way_comparable<T, std::strong_ordering>
void strong_compare(T a, T b) {
    auto result = a <=> b;
    // ...
}
```

这里，T 必须支持返回 std::strong_ordering 的三路比较，float 或 double 将不满足此要求（因为它们返回 std::partial_ordering）。

------

关系与区别

- **std::totally_ordered_with** 关注传统比较运算符（<, >, == 等）的完全有序性，适用于传统比较逻辑。
- **three_way_comparable** 关注三路比较运算符 <=> 的支持及其比较类别，适用于现代 C++ 的比较模型。
- 如果类型支持 <=> 并返回 std::strong_ordering 或 std::weak_ordering，它通常也满足 std::totally_ordered_with，因为 <=> 可以隐式定义传统比较运算符。

------

实际应用

1. **简化比较逻辑**：通过 <=>，只需定义一个运算符即可自动推导所有比较运算。
2. **模板约束**：使用这些概念确保模板函数或类只接受支持特定比较语义的类型。
3. **自定义类型**：为自定义类型实现 <=> 以支持现代比较模型。

示例：自定义类型

cpp

```cpp
#include <compare>
#include <iostream>

struct Point {
    int x, y;
    auto operator<=>(const Point& other) const = default; // 自动生成 strong_ordering
};

int main() {
    Point p1{1, 2}, p2{2, 1};
    std::cout << (p1 < p2) << '\n'; // 使用 <=> 自动推导 <
}
```

这里，Point 通过默认的 <=> 实现支持 std::three_way_comparable<Point, std::strong_ordering> 和 std::totally_ordered_with<Point, Point>。

------

总结

- **std::totally_ordered_with<T, U>**：确保 T 和 U 之间的比较是完全有序的，适用于传统比较运算符。
- **three_way_comparable<T, C>**：确保 T 支持三路比较 <=>，并返回指定类别的比较结果（strong_ordering, weak_ordering, 或 partial_ordering）。
- 两者结合 C++20 的 <=> 运算符，极大地简化了比较逻辑的实现和类型约束，适合现代 C++ 编程。

如果你有更具体的问题或需要进一步的代码示例，请告诉我！