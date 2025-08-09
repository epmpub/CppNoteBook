# 三路比较运算符

------

**基础知识**

- **三路比较运算符 <=>**:
  - C++20 引入，返回类型通常是 std::strong_ordering、std::weak_ordering 或 std::partial_ordering。
  - 提供 <、>、<=、>=、== 和 != 的完整比较。
  - 默认实现（= default）会逐成员比较（piecewise comparison）。
- **相等性比较 ==**:
  - 可以独立定义或默认生成。
  - 如果未显式定义，= default 的 <=> 会自动生成 == 和 !=。
- **关键函数**:
  - std::is_eq: 检查 <=> 结果是否表示相等。

------

**代码分解**

**1. 结构体 A**

cpp

```cpp
struct A {
    int x;
    int y;
    auto operator<=>(const A&) const = default;
};
```

- **operator<=> 默认实现**:
  - 使用 = default 表示编译器生成默认的三路比较。
  - 默认行为：对所有成员（x 和 y）进行逐成员三路比较。
  - 返回类型推导为 std::strong_ordering（因为 int 支持强序）。
- **生成的运算符**:
  - <、>、<=、>=：基于 <=> 自动生成。
  - == 和 !=：也自动生成，基于逐成员相等性比较。
- **行为**:
  - A a1{1, 2}; A a2{1, 2}; → a1 == a2 为 true。
  - A a1{1, 2}; A a2{1, 3}; → a1 < a2 为 true。
- **解释**: A 是**相等可比较（equality comparable）**的，因为默认 <=> 提供了完整的比较，包括相等性。

------

**2. 结构体 B**

cpp

```cpp
struct B {
    int x;
    int y;
    auto operator<=>(const B& other) const {
        auto cmp = x <=> other.x;
        if (!std::is_eq(cmp)) return cmp;
        return y <=> other.y;
    }
};
```

- **用户定义的 operator<=>**:
  - 自定义三路比较逻辑：
    1. 先比较 x 和 other.x，结果存入 cmp。
    2. 如果 cmp 不表示相等（!std::is_eq(cmp)），直接返回 cmp。
    3. 如果 x 相等，继续比较 y 和 other.y。
  - 这实现了字典序（lexicographical order）：先按 x 排序，x 相等时按 y 排序。
- **生成的运算符**:
  - <、>、<=、>=：基于 <=> 自动生成。
  - **== 和 != 未生成**：用户定义的 <=> 不会自动提供相等性比较。
- **行为**:
  - B b1{1, 2}; B b2{1, 2}; → b1 == b2 会编译失败（无 ==）。
  - B b1{1, 2}; B b2{1, 3}; → b1 < b2 为 true。
- **解释**: B **不是相等可比较的**，因为只定义了 <=>，没有 ==。

------

**3. 结构体 C**

cpp

```cpp
struct C {
    int x;
    int y;
    auto operator<=>(const C& other) const {
        auto cmp = x <=> other.x;
        if (!std::is_eq(cmp)) return cmp;
        return y <=> other.y;
    }
    bool operator==(const C&) const = default;
};
```

- **用户定义的 operator<=>**:
  - 与 B 相同，实现了字典序比较。
- **默认的 operator==**:
  - 使用 = default 表示编译器生成默认的相等性比较。
  - 默认行为：逐成员比较 x 和 y 是否相等。
- **生成的运算符**:
  - <、>、<=、>=：由 <=> 生成。
  - == 和 !=：由默认的 == 生成（!= 是 == 的反转）。
- **行为**:
  - C c1{1, 2}; C c2{1, 2}; → c1 == c2 为 true。
  - C c1{1, 2}; C c2{1, 3}; → c1 < c2 为 true。
- **解释**: C 是**相等可比较的**，因为显式定义了 ==，补充了 <=> 未提供的相等性。

------

**关键差异**

1. **A**:
   - 默认 <=> → 自动生成所有比较运算符（包括 == 和 !=）。
   - 相等可比较。
2. **B**:
   - 用户定义 <=> → 只生成 <、>、<=、>=，无 == 和 !=。
   - 非相等可比较。
3. **C**:
   - 用户定义 <=> + 默认 == → 提供所有比较运算符。
   - 相等可比较。

------

**中文解释**

**功能**

- 展示 C++20 中 <=> 和 == 的行为。
- 说明默认和用户定义比较的区别。

**结构体**

- **A**:
  - 默认 <=>，逐成员比较，自动生成所有运算符。
  - 可比较相等性（== 和 !=）。
- **B**:
  - 自定义 <=>，字典序比较，只生成顺序运算符。
  - 不可比较相等性（无 ==）。
- **C**:
  - 自定义 <=> + 默认 ==，结合顺序和相等性比较。
  - 可比较相等性。

**行为**

- A: 完全自动，适合简单类型。
- B: 自定义顺序，需手动补充 ==。
- C: 混合方式，灵活且完整。

------

**完整示例**

cpp

```cpp
#include <compare>
#include <iostream>

struct A { int x; int y; auto operator<=>(const A&) const = default; };
struct B {
    int x; int y;
    auto operator<=>(const B& other) const {
        auto cmp = x <=> other.x;
        if (!std::is_eq(cmp)) return cmp;
        return y <=> other.y;
    }
};
struct C {
    int x; int y;
    auto operator<=>(const C& other) const {
        auto cmp = x <=> other.x;
        if (!std::is_eq(cmp)) return cmp;
        return y <=> other.y;
    }
    bool operator==(const C&) const = default;
};

int main() {
    A a1{1, 2}, a2{1, 2};
    std::cout << (a1 == a2) << " " << (a1 < a2) << "\n"; // 1 0

    B b1{1, 2}, b2{1, 3};
    // std::cout << (b1 == b2) << "\n"; // 编译错误
    std::cout << (b1 < b2) << "\n"; // 1

    C c1{1, 2}, c2{1, 2};
    std::cout << (c1 == c2) << " " << (c1 < c2) << "\n"; // 1 0
}
```

------

**总结**

- **默认 <=>**（如 A）提供完整比较。
- **用户定义 <=>**（如 B）只提供顺序比较，需显式补充 ==。
- **混合方式**（如 C）结合自定义顺序和默认相等性，功能最全。

如果你有进一步问题或想深入某个部分，请告诉我！