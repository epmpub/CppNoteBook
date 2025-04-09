# std::bind

The utility for arbitrarily rebinding and reordering arguments of callable objects: std::bind.

The *std::bind* can be used to arbitrarily bind and remap arguments to (potentially multiple) callable objects.

Note that *std::bind_front* (C++20) and *std::bind_back* (C++26) offer better performance when you only need to bind leading or trailing arguments.

For complex use cases, a lambda might also offer better readability.

```C++
#include <functional>

// The constructed object stores the callable 
// and bound arguments
auto fill = [](int &x) { x = 42; };
auto b1 = std::bind(fill, 10);
b1(); // OK
// The bound argument stored in b1 now has value 42

int v = 0;
auto b2 = std::bind(fill, v);
b2(); // OK
// v == 0
// The bound argument stored in b2 now has value 42

auto b3 = std::bind(fill, std::ref(v));
b3(); // OK, b3 stores a reference to v
// v == 42

auto inc_print = [](int &x) { std::cout << x++ << '\n'; };
auto b4 = std::bind(inc_print, 42);
b4(); // prints 42
b4(); // prints 43
b4(); // prints 44

// Placeholders (ordinals/1-indexed) represent arguments 
// of the constructed object
auto fn1 = [](int a, int b, int c) { return (a+b)/c; };

using namespace std::placeholders;
auto b5 = std::bind(fn1, _1, _2, _3);
int r1 = b5(1, 2, 3);
// r1 == (1+2)/3 == 1

auto b6 = std::bind(fn1, _3, _1, _2);
int r2 = b6(1, 2, 3);
// r2 == (3+1)/2 == 2

// Placeholders are shared within nested bind expressions
auto fn = std::bind(
    std::plus<>{}, 
    _1, std::bind(std::multiplies<>{}, _1, _2)
);
// Same as:
auto cl = [](auto&& a, auto&& b) {
    return std::plus<>{}(
        std::forward<decltype(a)>(a),
        std::multiplies<>{}(
            std::forward<decltype(a)>(a), 
            std::forward<decltype(b)>(b)));
};
// fn(2,3) == cl(2,3) == 2+2*3

// Excess arguments are discarded (still evaluated)
auto b7 = std::bind([]{});
b7(10, 20, 30, std::cout << "discarded\n"); // OK
// prints "discarded"
```

这段代码展示了 C++ 中 <functional> 库中 std::bind 的用法，用于创建可调用对象并绑定参数。

它涵盖了绑定值、引用、占位符、嵌套绑定和冗余参数的处理。

以下是对代码的详细解释：

------

**代码结构**

1. **头文件**：
   - <functional>：提供 std::bind、std::ref 和 std::placeholders。
   - <iostream>：用于标准输出。
2. **主要内容**：
   - 使用 std::bind 绑定 lambda 函数和参数。
   - 展示值绑定、引用绑定的区别。
   - 使用占位符调整参数顺序。
   - 嵌套绑定和冗余参数处理。

------

**代码逐步解释**

**1. 绑定值（拷贝）**

cpp

```cpp
auto fill = [](int &x) { x = 42; };
auto b1 = std::bind(fill, 10);
b1(); // OK
```

- **fill**：一个 lambda，接受 int& 并将其设为 42。
- **std::bind(fill, 10)**：
  - 创建一个可调用对象 b1，将 10 绑定为 fill 的参数。
  - std::bind 默认按值传递参数，10 被拷贝到 b1 内部。
- **b1()**：
  - 调用时，fill 操作的是内部拷贝的 10，将其改为 42。
  - 外部看不到效果，因为 10 是常量。

------

**2. 绑定值（变量拷贝）**

cpp

```cpp
int v = 0;
auto b2 = std::bind(fill, v);
b2(); // OK, but no effect
std::cout << "v == " << v << '\n'; // v == 0
```

- **std::bind(fill, v)**：
  - 将 v 的值（0）拷贝到 b2 内部。
- **b2()**：
  - fill 修改的是 b2 内部的拷贝（从 0 改为 42）。
  - 外部的 v 不受影响，仍为 0。
- **输出**：v == 0。

------

**3. 绑定引用**

cpp

```cpp
auto b3 = std::bind(fill, std::ref(v));
b3(); // OK, b3 stores a reference to v
std::cout << "v == " << v << '\n'; // v == 42
```

- **std::ref(v)**：
  - 使用 std::reference_wrapper 将 v 按引用传递给 b3。
- **b3()**：
  - fill 修改的是 v 本身，从 0 变为 42。
- **输出**：v == 42。

------

**4. 修改绑定值**

cpp

```cpp
auto inc_print = [](int &x) { std::cout << x++ << '\n'; };
auto b4 = std::bind(inc_print, 42);
b4(); // prints 42
b4(); // prints 43
b4(); // prints 44
```

- **inc_print**：lambda 打印并递增 x。

- **std::bind(inc_print, 42)**：

  - 42 被拷贝到 b4 内部，成为可修改的副本。

- **b4()**：

  - 每次调用修改内部副本并打印。
  - 第一次：42，递增为 43。
  - 第二次：43，递增为 44。
  - 第三次：44，递增为 45。

- **输出**：

  ```text
  42
  43
  44
  ```

------

**5. 使用占位符**

cpp

```cpp
auto fn1 = [](int a, int b, int c) { return (a+b)/c; };
using namespace std::placeholders;
auto b5 = std::bind(fn1, _1, _2, _3);
int r1 = b5(1, 2, 3);
std::cout << "r1 == " << r1 << '\n'; // r1 == 1
```

- **_1, _2, _3**：
  - 占位符（1 索引），表示 b5 的第 1、2、3 个参数。
- **std::bind(fn1, _1, _2, _3)**：
  - 将 fn1 的参数按顺序绑定到调用时传入的参数。
- **b5(1, 2, 3)**：
  - a = 1, b = 2, c = 3。
  - (1 + 2) / 3 = 1。
- **输出**：r1 == 1。

------

**6. 调整参数顺序**

cpp

```cpp
auto b6 = std::bind(fn1, _3, _1, _2);
int r2 = b6(1, 2, 3);
std::cout << "r2 == " << r2 << '\n'; // r2 == 2
```

- **std::bind(fn1, _3, _1, _2)**：
  - a = _3, b = _1, c = _2。
- **b6(1, 2, 3)**：
  - a = 3, b = 1, c = 2。
  - (3 + 1) / 2 = 2。
- **输出**：r2 == 2。

------

**7. 嵌套绑定**

cpp

```cpp
auto fn = std::bind(
    std::plus<>{}, 
    _1, std::bind(std::multiplies<>{}, _1, _2)
);
auto cl = [](auto&& a, auto&& b) {
    return std::plus<>{}(
        std::forward<decltype(a)>(a),
        std::multiplies<>{}(
            std::forward<decltype(a)>(a), 
            std::forward<decltype(b)>(b)));
};
std::cout << "fn(2,3) == " << fn(2,3) << '\n'; // 8
std::cout << "cl(2,3) == " << cl(2,3) << '\n'; // 8
```

- **fn**：

  - 外层：std::plus<>{}（加法）。
  - 内层：std::bind(std::multiplies<>{}, _1, _2)（乘法）。
  - 等价于：a + (a * b)。

- **fn(2, 3)**：

  - _1 = 2, _2 = 3。
  - 2 + (2 * 3) = 2 + 6 = 8。

- **cl**：

  - 等价的 lambda，显式实现相同逻辑。

- **输出**：

  ```text
  fn(2,3) == 8
  cl(2,3) == 8
  ```

------

**8. 冗余参数**

cpp

```cpp
auto b7 = std::bind([]{});
b7(10, 20, 30, std::cout << "discarded\n"); // OK
```

- **std::bind([]{})**：

  - 绑定一个无参数的 lambda。

- **b7(10, 20, 30, std::cout << "discarded\n")**：

  - 多余参数被丢弃，但仍会被求值。
  - std::cout << "discarded\n" 执行，打印 "discarded"。

- **输出**：

  ```text
  discarded
  ```

------

**关键技术点**

1. **std::bind**：
   - 创建可调用对象，绑定参数或调整调用签名。
   - 默认按值绑定，需用 std::ref 绑定引用。
2. **占位符**：
   - std::placeholders::_N 表示调用时的第 N 个参数。
   - 可重排或重复使用参数。
3. **值 vs 引用**：
   - 值绑定修改内部副本，引用绑定修改外部变量。
4. **嵌套绑定**：
   - 支持复杂逻辑组合，占位符在嵌套中共享。
5. **冗余参数**：
   - 多余参数被忽略，但表达式仍求值。

------

**输出总结**

```text
v == 0
v == 42
42
43
44
r1 == 1
r2 == 2
fn(2,3) == 8
cl(2,3) == 8
discarded
```

------

**总结**

- **std::bind**：灵活绑定参数，支持值、引用和占位符。
- **用途**：调整函数签名、预设参数、创建回调。
- **注意**：C++11 引入，但 C++14 后的 lambda 常更简洁。

如果你有具体问题（例如嵌套绑定的细节），欢迎提问！