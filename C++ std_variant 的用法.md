#  std::variant 的用法

这段代码展示了 C++17 中，以及如何处理其初始化和空状态（empty state）。

std::variant 是一个类型安全的联合体（union），可以存储多个指定类型中的一种。

代码中涉及了默认构造、显式初始化和使用 std::monostate 表示空状态的场景。

------

cpp

```cpp
#include <variant>

struct X {
    X(int v) : v_(v) {}
    int v_;
};

struct Y {
    Y(double v) : v_(v) {}
    double v_;
};

// X, Y are non-default constructible
// std::variant<X,Y> a; wouldn't not compile, can't default construct

std::variant<X,Y> a = Y{20}; // OK
// a.index() == 1

std::variant<std::monostate,X,Y> b; // OK, semantically "empty"
// b.index() == 0, std::holds_alternative<std::monostate>(b) == true

std::variant<int,double> c;
std::variant<int,double> d{0};
// c == d

// to distinguish an empty state:
std::variant<std::monostate,int,double> e;
std::variant<std::monostate,int,double> f{0};
// e != f
```

------

逐步解释

**部分 1：非默认构造类型**

cpp

```cpp
struct X {
    X(int v) : v_(v) {}
    int v_;
};

struct Y {
    Y(double v) : v_(v) {}
    double v_;
};

// std::variant<X,Y> a; // 编译失败
```

- **X 和 Y**：
  - 自定义类型，没有默认构造函数（因定义了带参数的构造函数）。
- **std::variant<X,Y> a;**：
  - std::variant 的默认构造函数会尝试默认构造其第一个类型（这里是 X）。
  - 因为 X 没有默认构造函数，编译失败。

**初始化 a**

cpp

```cpp
std::variant<X,Y> a = Y{20};
// a.index() == 1
```

- **a = Y{20}**：
  - 显式初始化 a 为 Y 类型的值（Y{20}）。
  - std::variant 存储 Y，a.index() 返回 1（索引从 0 开始，X 是 0，Y 是 1）。
- **验证**：
  - std::get<Y>(a).v_ == 20。
  - std::holds_alternative<Y>(a) == true。

------

**部分 2：使用 std::monostate 表示空状态**

cpp

```cpp
std::variant<std::monostate,X,Y> b;
// b.index() == 0, std::holds_alternative<std::monostate>(b) == true
```

- **std::monostate**：
  - 一个空类型（无数据），用于表示“无值”状态。
  - 可默认构造。
- **std::variant<std::monostate,X,Y> b;**：
  - 默认构造时，std::variant 初始化为第一个类型（std::monostate）。
  - b.index() == 0，表示当前持有 std::monostate。
- **语义**：
  - 表示 b 是“空的”，类似于 std::optional 的未赋值状态。

------

**部分 3：默认构造与显式初始化**

cpp

```cpp
std::variant<int,double> c;
std::variant<int,double> d{0};
// c == d
```

- **std::variant<int,double> c;**：
  - 默认构造，初始化为第一个类型（int）的默认值（0）。
  - c.index() == 0，std::get<int>(c) == 0。
- **std::variant<int,double> d{0};**：
  - 显式初始化为 int 类型的值 0。
  - d.index() == 0，std::get<int>(d) == 0。
- **c == d**：
  - 两个 variant 的类型（int）和值（0）相同，相等。

**问题**

- 默认构造的 c 是 int{0}，无法区分“空状态”和“值为 0”的情况。

------

**部分 4：使用 std::monostate 区分空状态**

cpp

```cpp
std::variant<std::monostate,int,double> e;
std::variant<std::monostate,int,double> f{0};
// e != f
```

- **std::variant<std::monostate,int,double> e;**：
  - 默认构造，初始化为 std::monostate。
  - e.index() == 0，表示“空状态”。
- **std::variant<std::monostate,int,double> f{0};**：
  - 显式初始化为 int{0}。
  - f.index() == 1（std::monostate 是 0，int 是 1）。
- **e != f**：
  - e 和 f 的类型不同（std::monostate vs int），不相等。
- **优势**：
  - 使用 std::monostate 明确区分了“空”（index == 0）和“有值”（index > 0）。

------

输出验证（假设打印）

如果添加打印代码：

cpp

```cpp
#include <variant>
#include <iostream>

int main() {
    std::variant<X,Y> a = Y{20};
    std::cout << "a.index(): " << a.index() << "\n";

    std::variant<std::monostate,X,Y> b;
    std::cout << "b.index(): " << b.index() << "\n";

    std::variant<int,double> c;
    std::variant<int,double> d{0};
    std::cout << "c == d: " << (c == d) << "\n";

    std::variant<std::monostate,int,double> e;
    std::variant<std::monostate,int,double> f{0};
    std::cout << "e == f: " << (e == f) << "\n";

    return 0;
}
```

输出

```text
a.index(): 1
b.index(): 0
c == d: 1
e == f: 0
```

------

关键点分析

1. **std::variant 的默认构造**：
   - 默认构造第一个类型。
   - 若第一个类型不可默认构造，则编译失败。
2. **std::monostate**：
   - 一个空占位符，用于表示“无值”。
   - 总是可默认构造，适合作为第一个类型。
3. **初始化**：
   - 显式初始化（如 Y{20}、0）覆盖默认行为。
4. **比较**：
   - == 比较类型和值，必须相同才相等。

------

时间复杂度

- **构造**：O(1)，取决于类型的构造成本。
- **访问（如 index()）**：O(1)。
- **比较（如 ==）**：O(1)，若类型支持常量时间比较。

------

使用场景

1. **非默认构造类型**：
   - 使用显式初始化或 std::monostate。
2. **空状态表示**：
   - 用 std::monostate 区分“无值”和“有值”。
3. **类型安全替代 union**：
   - std::variant 提供安全的类型切换。

------

注意事项

1. **C++17 要求**：
   - 需要 -std=c++17 或更高版本。
2. **异常**：
   - 若访问错误类型（std::get），抛出 std::bad_variant_access。
3. **初始化检查**：
   - 使用 index() 或 holds_alternative 验证状态。

------

总结

这段代码展示了：

- **std::variant<X,Y>**：需显式初始化（如 Y{20}），否则因无默认构造而失败。
- **std::monostate**：用于表示空状态，解决默认构造问题。
- **std::variant<int,double>**：默认构造为 int{0}，无法区分空状态。
- **std::monostate,int,double>**：明确区分空（monostate）和值（如 int{0}）。 std::variant 结合 std::monostate 提供了灵活的状态管理，是现代 C++ 的重要工具。如果你有具体问题或想扩展代码，请告诉我！