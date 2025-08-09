# std::tie

```C++
#include <tuple>
#include <print>

struct Coord {
    int x;
    int y;

    // We can use std::tie to simplify some operations
    bool operator<(const Coord& other) const {
        return std::tie(x, y) < std::tie(other.x, other.y);
    }
    bool operator==(const Coord& other) const {
        return std::tie(x, y) == std::tie(other.x, other.y);
    }
    friend void swap(Coord& l, Coord& r) {
        std::swap(std::tie(l.x, l.y), std::tie(r.x, r.y));
    }
};

std::pair<double,int> fun() { return {4.2, 42}; }

int main() {
    int x{};
    const double y{};

    // Creates a tuple of lvalue references
    auto t1 = std::tie(x, y);
    // decltype(t1) == std::tuple<int&, const double&>

    static_assert(std::is_same_v<decltype(t1), std::tuple<int&, const double&>>);

    // std::tie can be used as a less elegant version of structured bindings
    double i{}; int j{};
    std::tie(i,j) = fun();
    // i == 4.2, j == 42

    std::println("i == {}, j == {}", i, j);

    // std::ignore can be combined with std::tie to skip over some fields
    std::tie(std::ignore, j) = std::pair{0.0, 7};
    // i == 4.2, j == 7

    std::println("i == {}, j == {}", i, j);

    // We can use std::tie to re-use structured binding identifiers
    auto v = std::pair{4.2, 42};
    auto [m, n] = v; // copy of v
    // m == 4.2, n == 42

    std::println("v == [{}, {}], m == {}, n == {}", v.first, v.second, m, n);

    std::tie(m, n) = std::pair{1.1, 2};
    // m == 1.1, n == 2
    std::println("v == [{}, {}], m == {}, n == {}", v.first, v.second, m, n);

    auto& [p, q] = v; // in this case we have a reference to v
    std::tie(p, q) = std::pair{1.1, 2};
    // v == {1.1, 2}, p == 1.1, q == 2
    std::println("v == [{}, {}], p == {}, q == {}", v.first, v.second, p, q);

    Coord a{1, 1}, b{1, 2};
    std::println("(a < b) == {}", a < b);
    std::println("(a == b) == {}", a == b);
    swap(a, b);

    std::println("a == [{},{}], b == [{},{}]", a.x, a.y, b.x, b.y);
}
```



这段代码展示了 C++ 中 <tuple> 头文件提供的 std::tie 的多种用法，包括简化比较操作、赋值、交换，以及与结构化绑定的结合。代码通过 Coord 结构体和 main 函数中的示例展示了这些功能。以下是逐步解释。

------

代码概览

- 使用 std::tie 在 Coord 中实现比较和交换。
- 在 main 中展示 std::tie 的赋值、忽略字段、与结构化绑定的配合等用法。

------

关键组件

1. **头文件**

cpp

```cpp
#include <tuple>
#include <print>
```

- <tuple>：提供 std::tie 和 std::ignore。
- <print>：C++23 的 std::println 用于输出。
- **Coord 结构体**

cpp

```cpp
struct Coord {
    int x;
    int y;

    bool operator<(const Coord& other) const {
        return std::tie(x, y) < std::tie(other.x, other.y);
    }
    bool operator==(const Coord& other) const {
        return std::tie(x, y) == std::tie(other.x, other.y);
    }
    friend void swap(Coord& l, Coord& r) {
        std::swap(std::tie(l.x, l.y), std::tie(r.x, r.y));
    }
};
```

- **std::tie(x, y)**：
  - 创建一个 std::tuple<int&, int&>，绑定到 x 和 y 的引用。
- **operator< 和 operator==**：
  - 使用 std::tie 将成员打包为元组，自动实现字典序比较。
  - 简化多字段比较代码。
- **swap**：
  - 使用 std::swap 交换两个元组，实际交换 x 和 y。
- **fun 函数**

cpp

```cpp
std::pair<double,int> fun() { return {4.2, 42}; }
```

- 返回一个 std::pair，用于测试赋值。
- **main 函数**

**创建引用元组**

cpp

```cpp
int x{};
const double y{};
auto t1 = std::tie(x, y);
static_assert(std::is_same_v<decltype(t1), std::tuple<int&, const double&>>);
```

- **std::tie(x, y)**：
  - 创建 std::tuple<int&, const double&>，绑定到 x 和 y。
- **类型验证**：
  - static_assert 确认 t1 的类型。

**赋值操作**

cpp

```cpp
double i{}; int j{};
std::tie(i,j) = fun();
std::println("i == {}, j == {}", i, j);
```

- **std::tie(i,j) = fun()**：

  - fun() 返回 {4.2, 42}。
  - 赋值给 i 和 j，类似结构化绑定。

- **输出**：

  ```text
  i == 4.2, j == 42
  ```

**忽略字段**

cpp

```cpp
std::tie(std::ignore, j) = std::pair{0.0, 7};
std::println("i == {}, j == {}", i, j);
```

- **std::ignore**：

  - 占位符，忽略对应字段（0.0）。

- **结果**：

  - j 更新为 7，i 保持 4.2。

- **输出**：

  ```text
  i == 4.2, j == 7
  ```

**与结构化绑定结合**

cpp

```cpp
auto v = std::pair{4.2, 42};
auto [m, n] = v;
std::println("v == [{}, {}], m == {}, n == {}", v.first, v.second, m, n);
std::tie(m, n) = std::pair{1.1, 2};
std::println("v == [{}, {}], m == {}, n == {}", v.first, v.second, m, n);
```

- **auto [m, n] = v**：

  - 复制 v 的值，m = 4.2，n = 42。

- **std::tie(m, n) = ...**：

  - 更新 m 和 n 为 1.1 和 2，v 不变。

- **输出**：

  ```text
  v == [4.2, 42], m == 4.2, n == 42
  v == [4.2, 42], m == 1.1, n == 2
  ```

**绑定引用**

cpp

```cpp
auto& [p, q] = v;
std::tie(p, q) = std::pair{1.1, 2};
std::println("v == [{}, {}], p == {}, q == {}", v.first, v.second, p, q);
```

- **auto& [p, q] = v**：

  - p 和 q 是 v.first 和 v.second 的引用。

- **std::tie(p, q) = ...**：

  - 更新 p 和 q，同时修改 v。

- **输出**：

  ```text
  v == [1.1, 2], p == 1.1, q == 2
  ```

**Coord 测试**

cpp

```cpp
Coord a{1, 1}, b{1, 2};
std::println("(a < b) == {}", a < b);
std::println("(a == b) == {}", a == b);
swap(a, b);
std::println("a == [{},{}], b == [{},{}]", a.x, a.y, b.x, b.y);
```

- **比较**：

  - a < b：(1, 1) < (1, 2) 为真。
  - a == b：不相等。

- **swap**：

  - 交换 a 和 b。

- **输出**：

  ```text
  (a < b) == true
  (a == b) == false
  a == [1,2], b == [1,1]
  ```

------

为什么这样工作？

1. **std::tie**：
   - 创建引用元组，简化多字段操作。
2. **std::ignore**：
   - 忽略赋值中的部分字段。
3. **与结构化绑定**：
   - std::tie 提供赋值能力，补充绑定的局限。
4. **Coord**：
   - std::tie 自动实现字典序逻辑。

------

输出

```text
i == 4.2, j == 42
i == 4.2, j == 7
v == [4.2, 42], m == 4.2, n == 42
v == [4.2, 42], m == 1.1, n == 2
v == [1.1, 2], p == 1.1, q == 2
(a < b) == true
(a == b) == false
a == [1,2], b == [1,1]
```

------

使用场景

- **多字段比较**：
  - 简化 operator< 和 operator==。
- **赋值**：
  - 从 pair 或 tuple 提取值。
- **交换**：
  - 高效交换结构体成员。

------

总结

- std::tie 在 Coord 中实现比较和交换。
- 在 main 中用于赋值、忽略字段和更新绑定。
- 代码展示了 std::tie 的多功能性和实用性。