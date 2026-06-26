# C++ 20 Concepts equality_comparable_with

在 C++20 及之后的版本中，`equality_comparable_with` 是 Concepts 库的一部分，用于约束**可以互相进行相等性比较**的类型。它检查两个类型是否能用 `==` 和 `!=` 进行比较，并且保证这种比较在逻辑上是一致的。

---

### **`equality_comparable_with` 的定义**
该概念定义在 `<concepts>` 头文件中，要求：
1. 两种类型都能用 `==` 和 `!=` 进行比较；
2. 比较是**对称的**（如果 `a == b` 合法，那么 `b == a` 也必须合法）；
3. 比较是**可传递的**（如果 `a == b` 且 `b == c`，那么 `a == c` 应该成立）。

---

### **语法**
```cpp
template <class T, class U>
concept equality_comparable_with =
    std::equality_comparable<T> &&       // T 必须支持自身比较
    std::equality_comparable<U> &&       // U 必须支持自身比较
    requires(const std::remove_reference_t<T>& t, const std::remove_reference_t<U>& u) {
        { t == u } -> std::convertible_to<bool>;  // T 和 U 可以互相用 == 比较
        { u == t } -> std::convertible_to<bool>;  // 反向也必须支持
        { t != u } -> std::convertible_to<bool>;  // 支持 != 比较
        { u != t } -> std::convertible_to<bool>;  // 反向 != 比较
    };
```

---

### **使用示例**
```cpp
#include <concepts>
#include <iostream>

// 约束 T 和 U 必须可以互相比较
template <typename T, typename U>
requires std::equality_comparable_with<T, U>
bool check_equal(const T& a, const U& b) {
    return a == b;
}

struct A {
    int x;
    bool operator==(const A& other) const { return x == other.x; }
    bool operator==(int val) const { return x == val; }  // A 可以和 int 比较
};

int main() {
    A a1{5}, a2{5};
    std::cout << check_equal(a1, a2) << '\n';  // true (A == A)
    std::cout << check_equal(a1, 5) << '\n';   // true (A == int)
    // std::cout << check_equal(5, a1) << '\n'; // 错误：int 不知道怎么和 A 比较
}
```

---

### **关键点**
1. **对称性要求**：
   - 如果 `T` 能和 `U` 比较，那么 `U` 也必须能和 `T` 比较。
   - 上面的例子中，`A` 定义了 `operator==(int)`，但 `int` 没有定义 `operator==(A)`，所以 `check_equal(5, a1)` 会失败。
   - 解决方法：
     - 额外定义一个自由函数 `bool operator==(int, const A&)`；
     - 或者确保两种类型都支持互相比较。

2. **常见用途**：
   - 在泛型编程中约束类型，确保它们可以互相比较；
   - 提高模板代码的类型安全性。

3. **相关概念**：
   - `std::equality_comparable<T>`（检查一个类型是否能和自身比较）；
   - `std::totally_ordered_with<T, U>`（扩展到 `<`, `>`, `<=`, `>=` 等比较）。

---

### **修复不对称比较（示例）**
要让 `A` 和 `int` 完全支持双向比较：
```cpp
bool operator==(int val, const A& a) { return a == val; }  // 现在 int == A 也合法

int main() {
    A a1{5};
    std::cout << check_equal(5, a1) << '\n';  // 现在可以运行
}
```

---

### **总结**
`equality_comparable_with` 确保两种类型可以互相进行 `==` 和 `!=` 比较，适用于需要**约束类型可互比**的泛型编程场景。如果只支持单向比较，编译时会报错，帮助开发者提前发现问题。