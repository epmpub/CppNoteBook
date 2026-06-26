# std::totally_ordered 

在 C++20 中，std::totally_ordered 是一个概念（concept），定义在 <concepts> 头文件中，用于约束一个类型 T，确保它自身支持完全有序（total ordering）的比较操作。

这意味着类型 T 的对象可以通过比较运算符（<, >, <=, >=, ==, !=）进行比较，并且这些比较满足完全有序的数学性质。

------

定义

std::totally_ordered<T> 要求：

1. 类型 T 支持所有比较运算符：==, !=, <, >, <=, >=。
2. 这些比较运算符的结果满足完全有序的性质：
   - **反对称性**：若 a < b，则 !(b < a)。
   - **传递性**：若 a < b 且 b < c，则 a < c。
   - **完全性**：对于任意 a 和 b，要么 a < b，要么 b < a，要么 a == b。
3. 比较操作必须是类型安全的，结果是可预测的。

关系

std::totally_ordered<T> 是 std::totally_ordered_with<T, T> 的特例，专注于单一类型 T 自身的比较，而 std::totally_ordered_with<T, U> 约束两个类型 T 和 U 之间的比较。

------

用法

std::totally_ordered 通常用于模板编程中，以确保模板参数类型支持完全有序的比较。例如：

cpp

```cpp
#include <concepts>
#include <iostream>

template<typename T>
requires std::totally_ordered<T>
void compare(T a, T b) {
    if (a < b) std::cout << a << " < " << b << '\n';
    else if (a > b) std::cout << a << " > " << b << '\n';
    else std::cout << a << " == " << b << '\n';
}

int main() {
    compare(5, 10);     // 合法：int 满足 totally_ordered
    compare(3.14, 2.0); // 不合法：double 不满足 totally_ordered（因 NaN 破坏完全性）
}
```

在这个例子中，compare 函数要求类型 T 满足 std::totally_ordered，因此 int 可以通过编译，而 double 由于可能包含 NaN（不可比较）而不满足要求。

------

与三路比较的关系

在 C++20 中，三路比较运算符 <=>（spaceship operator）可以自动推导传统比较运算符。如果一个类型 T 定义了 <=> 运算符，并且返回 std::strong_ordering 或 std::weak_ordering，它通常也满足 std::totally_ordered。

示例：使用 <=>

cpp

```cpp
#include <compare>
#include <concepts>
#include <iostream>

struct Point {
    int x, y;
    auto operator<=>(const Point& other) const = default; // 自动生成 strong_ordering
};

template<typename T>
requires std::totally_ordered<T>
void compare(T a, T b) {
    if (a < b) std::cout << "a < b\n";
    else if (a > b) std::cout << "a > b\n";
    else std::cout << "a == b\n";
}

int main() {
    Point p1{1, 2}, p2{2, 1};
    compare(p1, p2); // 合法：Point 满足 totally_ordered
}
```

这里，Point 通过默认的 <=> 实现支持完全有序比较，满足 std::totally_ordered<Point>。

------

注意事项

1. **浮点数不满足 std::totally_ordered**：
   - 类型如 float 和 double 由于存在 NaN（Not-a-Number），不满足完全有序的要求（NaN 与任何值比较都不成立）。
   - 因此，std::totally_ordered<float> 或 std::totally_ordered<double> 为 false。
2. **自定义类型**：
   - 自定义类型需要显式定义比较运算符或 <=> 运算符，并确保比较满足完全有序。
   - 使用 = default 的 <=> 通常生成 std::strong_ordering，满足 std::totally_ordered。
3. **与 std::three_way_comparable 的关系**：
   - 如果类型 T 满足 std::three_way_comparable<T, std::strong_ordering> 或 std::three_way_comparable<T, std::weak_ordering>，它通常也满足 std::totally_ordered<T>。
   - 但 std::three_way_comparable<T, std::partial_ordering>（如 float）不一定满足 std::totally_ordered<T>。

------

实际应用

1. **模板约束**：在泛型编程中，确保类型支持完全有序比较，适用于排序、查找等算法。
2. **类型安全**：通过概念约束，避免在不支持完全有序的类型上进行比较操作。
3. **简化比较逻辑**：结合 <=> 运算符，减少手动定义多个比较运算符的工作量。

------

总结

- **std::totally_ordered<T>** 约束类型 T 自身支持完全有序的比较操作，适用于传统比较运算符（<, >, ==, 等）。
- 它要求比较满足反对称性、传递性和完全性，适用于如 int、自定义类型（定义了 <=> 或比较运算符）等，但不适用于 float 或 double（因 NaN）。
- 在 C++20 中，结合 <=> 和 std::three_way_comparable，可以更方便地实现和验证完全有序的类型。

如果你有更具体的问题或需要进一步的代码示例，请告诉我！