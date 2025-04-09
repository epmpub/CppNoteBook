

# C++23 tuple-like and pair-like types

C++23 formalized the concepts of tuple-like and pair-like types.

The set of tuple-like types is currently: *std::array*, *std::pair*, *std::tuple*, *std::ranges::subrange* and *std::complex* (since C++26).

Pair-like types are tuple-like types that have two elements.

All tuple-like types support a base set of operations:

- assignment
- comparisons
- *std::apply*, *std::make_from_tuple*
- *std::tuple_size*, *std::tuple_element*
- *std::tuple_cat*

```C++
#include <tuple>
#include <array>

#include <print>

int main() {
    auto a = std::tuple{1, 3.4, "Hello World"};
    auto b = std::pair{7, 1.7};
    auto c = std::array{1,2,3};
    auto d = std::tuple{9,8,7};

    // Comparison
    // c < d == true
    std::println("c < d == {}", c < d);

    // Assignment
    d = c; // array -> tuple
    // d == {1,2,3}

    std::println("d == [{},{},{}]", std::get<0>(d), std::get<1>(d), std::get<2>(d));

    auto fn = [](int a, double b) { return a + b; };

    // std::apply: call a function with the elements 
    // of the tuple-like object as arguments
    auto r = std::apply(fn, b);
    // decltype(r) == double, r == 8.7

    std::println("r == {}", r);

    // std::make_from_tuple: construct a object with the elements
    // of the tuple-like object as arguments of the constructor
    struct X { int x; int y; int z; };
    auto obj = std::make_from_tuple<X>(c);
    // obj == {1, 2, 3}

    std::println("obj == {{{},{}, {}}}", obj.x, obj.y, obj.z);

    // std::tuple_element: the i-th element type
    using e_t = std::tuple_element_t<1, decltype(a)>;
    // e_t == double

    static_assert(std::is_same_v<e_t,double>);

    // std::tuple_size: number of elements
    size_t sz = std::tuple_size<decltype(b)>{};
    // sz == 2

    std::println("sz == {}", sz);

    // std::tuple_cat:
    // concatenate two tuple-like objects into a tuple
    auto cat = std::tuple_cat(b, c);
    // decltype(cat) == std::tuple<int, double, int, int, int>
    // cat == {7, 1.7, 1, 2, 3}

    std::println("cat == [{},{},{},{},{}]", 
        std::get<0>(cat), std::get<1>(cat), std::get<2>(cat),
        std::get<3>(cat), std::get<4>(cat));
}
```

这段代码展示了 C++ 中 <tuple> 和 <array> 库的使用，涉及元组（std::tuple）、对组（std::pair）、数组（std::array）的多种操作，包括比较、赋值、函数调用、构造对象、类型查询和拼接等功能。以下是对代码的详细解释：

------

**代码结构**

1. **头文件**：
   - <tuple>：提供 std::tuple、std::pair 和相关工具（如 std::apply）。
   - <array>：提供 std::array。
   - <print>：C++23 的 std::println 用于格式化输出。
2. **主要内容**：
   - 创建不同类型的元组和数组。
   - 展示比较、赋值和元组操作。

------

**代码逐步解释**

**1. 初始化**

cpp

```cpp
auto a = std::tuple{1, 3.4, "Hello World"};
auto b = std::pair{7, 1.7};
auto c = std::array{1, 2, 3};
auto d = std::tuple{9, 8, 7};
```

- **a**：std::tuple<int, double, const char*>，包含 {1, 3.4, "Hello World"}。
- **b**：std::pair<int, double>，包含 {7, 1.7}。
- **c**：std::array<int, 3>，包含 {1, 2, 3}。
- **d**：std::tuple<int, int, int>，包含 {9, 8, 7}。

------

**2. 比较**

cpp

```cpp
std::println("c < d == {}", c < d);
```

- **c < d**：

  - std::array 和 std::tuple 支持比较。
  - 按字典序（lexicographical order）比较元素：
    - c[0] = 1 < d[0] = 9，为真，比较结束。
  - 结果：true。

- **输出**：

  ```text
  c < d == true
  ```

------

**3. 赋值**

cpp

```cpp
d = c; // array -> tuple
std::println("d == [{},{},{}]", std::get<0>(d), std::get<1>(d), std::get<2>(d));
```

- **d = c**：

  - 将 std::array<int, 3> 赋值给 std::tuple<int, int, int>。
  - 类型和大小匹配，元素逐个复制。
  - d 从 {9, 8, 7} 变为 {1, 2, 3}。

- **std::get<N>**：

  - 访问元组的第 N 个元素（0-based）。

- **输出**：

  ```text
  d == [1,2,3]
  ```

------

**4. std::apply 调用函数**

cpp

```cpp
auto fn = [](int a, double b) { return a + b; };
auto r = std::apply(fn, b);
std::println("r == {}", r);
```

- **fn**：

  - Lambda 函数，接受 int 和 double，返回它们的和。

- **std::apply**：

  - C++17 引入，将元组的元素解包作为函数参数。
  - b 是 std::pair<int, double>{7, 1.7}。
  - 调用：fn(7, 1.7)，结果为 7 + 1.7 = 8.7。

- **r**：

  - 类型为 double，值 8.7。

- **输出**：

  ```text
  r == 8.7
  ```

------

**5. std::make_from_tuple 构造对象**

cpp

```cpp
struct X { int x; int y; int z; };
auto obj = std::make_from_tuple<X>(c);
std::println("obj == {{{},{}, {}}}", obj.x, obj.y, obj.z);
```

- **X**：

  - 结构体，含三个 int 成员。

- **std::make_from_tuple**：

  - C++17 引入，用元组元素构造对象。
  - c 是 std::array<int, 3>{1, 2, 3}。
  - 调用：X{1, 2, 3}，初始化 obj。

- **obj**：

  - obj.x = 1, obj.y = 2, obj.z = 3。

- **输出**：

  ```text
  obj == {1,2, 3}
  ```

------

**6. std::tuple_element 获取元素类型**

cpp

```cpp
using e_t = std::tuple_element_t<1, decltype(a)>;
static_assert(std::is_same_v<e_t, double>);
```

- **std::tuple_element_t**：
  - C++17 别名模板，返回元组第 N 个元素的类型。
  - decltype(a) 是 std::tuple<int, double, const char*>。
  - 1 表示第 2 个元素（0-based），类型为 double。
- **static_assert**：
  - 验证 e_t 是 double。

------

**7. std::tuple_size 获取元素数量**

cpp

```cpp
size_t sz = std::tuple_size<decltype(b)>{};
std::println("sz == {}", sz);
```

- **std::tuple_size**：

  - 返回元组的元素个数。
  - decltype(b) 是 std::pair<int, double>。
  - std::tuple_size<>::value 为 2，{} 创建临时对象访问。

- **sz**：

  - 值 2。

- **输出**：

  ```text
  sz == 2
  ```

------

**8. std::tuple_cat 拼接元组**

cpp

```cpp
auto cat = std::tuple_cat(b, c);
std::println("cat == [{},{},{},{},{}]", 
    std::get<0>(cat), std::get<1>(cat), std::get<2>(cat),
    std::get<3>(cat), std::get<4>(cat));
```

- **std::tuple_cat**：

  - 拼接多个元组或类元组对象。
  - b 是 std::pair<int, double>{7, 1.7}。
  - c 是 std::array<int, 3>{1, 2, 3}。
  - 结果：std::tuple<int, double, int, int, int>{7, 1.7, 1, 2, 3}。

- **std::get<N>**：

  - 访问拼接后的元素。

- **输出**：

  ```text
  cat == [7,1.7,1,2,3]
  ```

------

**关键技术点**

1. **std::tuple 和 std::pair**：
   - 异构容器，支持不同类型。
2. **std::array**：
   - 同构固定大小容器，可与元组互操作。
3. **std::apply**：
   - 解包元组调用函数。
4. **std::make_from_tuple**：
   - 用元组构造对象。
5. **元组工具**：
   - tuple_element_t：获取类型。
   - tuple_size：获取大小。
   - tuple_cat：拼接元组。

------

**输出总结**

```text
c < d == true
d == [1,2,3]
r == 8.7
obj == {1,2, 3}
sz == 2
cat == [7,1.7,1,2,3]
```

------

**可能的改进或注意事项**

1. **类型显式性**：
   - 可显式声明元组类型（如 std::tuple<int, double, const char*>）。
2. **错误处理**：
   - std::get 在越界时抛异常，可添加检查。
3. **格式优化**：
   - std::println 可结合循环简化 cat 输出。

------

**总结**

- **功能**：展示元组和数组的比较、赋值和工具函数。
- **现代特性**：结合 C++17 和 C++23 特性。
- **用途**：处理异构数据、解包调用等场景。

如果你有具体问题（例如 std::apply 的其他用法），欢迎提问！