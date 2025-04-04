# 隐式Lambda转换到函数指针

Forcing conversion of a lambda to a function pointer



```C++
#include <iostream>

template <typename Callback>
struct X {
    Callback call;
};

// Each lambda expression produces a unique distinct type.
X x1([]() { std::cout << "Hello World!\n"; });
X x2([]() { std::cout << "Hello Universe!\n"; });
static_assert(!std::is_same_v<decltype(x1),decltype(x2)>);

// unary + operator does not work on lambdas but does work on pointers
// it forces the lambda -> function pointer implicit conversion
X x3(+[]() { std::cout << "Hello World!\n"; });
X x4(+[]() { std::cout << "Hello Universe!\n"; });
static_assert(std::is_same_v<decltype(x3),decltype(x4)>);

// static_cast approach
X x5(static_cast<void(*)()>([]() { std::cout << "Same old.\n"; }));
static_assert(std::is_same_v<decltype(x4), decltype(x5)>);
```



让我逐步解释这段代码，分析其背后的概念和行为。

代码结构

这段代码定义了一个模板结构体 X，它接受一个类型参数 Callback，并有一个成员变量 call，类型为 Callback。然后通过不同的方式（lambda 表达式、单目加号 + 操作符和 static_cast）创建了多个 X 实例，并使用 static_assert 检查这些实例的类型是否相同。

------

逐步分析

1. 基本模板定义

cpp

```cpp
template <typename Callback>
struct X {
    Callback call;
};
```

- X 是一个模板结构体，Callback 是一个类型参数。
- call 是 X 的成员变量，它的类型由模板参数 Callback 决定。

------

2. Lambda 表达式生成唯一类型

cpp

```cpp
X x1([]() { std::cout << "Hello World!\n"; });
X x2([]() { std::cout << "Hello Universe!\n"; });
static_assert(!std::is_same_v<decltype(x1),decltype(x2)>);
```

- 这里用两个 lambda 表达式实例化了 X：
  - x1 的 lambda 是 []() { std::cout << "Hello World!\n"; }
  - x2 的 lambda 是 []() { std::cout << "Hello Universe!\n"; }
- **关键点**：在 C++ 中，每个 lambda 表达式都会生成一个独一无二的闭包类型，即使它们的签名和实现看起来类似。这是因为 lambda 的类型是由编译器生成的匿名类，且每个 lambda 都被视为不同的类型。
- decltype(x1) 和 decltype(x2) 返回 X<某种闭包类型>，但由于两个 lambda 的闭包类型不同，X<闭包类型1> 和 X<闭包类型2> 也是不同的类型。
- static_assert(!std::is_same_v<decltype(x1),decltype(x2)>) 检查 x1 和 x2 的类型是否不同，结果为真（即它们确实是不同类型），所以编译通过。

------

3. 单目加号 + 操作符的作用

cpp

```cpp
X x3(+[]() { std::cout << "Hello World!\n"; });
X x4(+[]() { std::cout << "Hello Universe!\n"; });
static_assert(std::is_same_v<decltype(x3),decltype(x4)>);
```

- 这里对 lambda 使用了单目加号 + 操作符：+[]() { ... }。
- **关键点**：lambda 表达式本身不能直接应用 + 操作符，但 C++ 允许 lambda 隐式转换为函数指针（如果它没有捕获变量）。单目 + 操作符会触发这种转换。
  - 例如，[]() { std::cout << "Hello World!\n"; } 被转换为类型 void(*)()（指向无参数、无返回值的函数的指针）。
- 因此：
  - x3 的 Callback 类型是 void(*)()。
  - x4 的 Callback 类型也是 void(*)()。
- 因为两个 lambda 都被转换为相同的函数指针类型 void(*)()，所以 decltype(x3)（即 X<void(*)()>） 和 decltype(x4)（即 X<void(*)()>） 是相同的类型。
- static_assert(std::is_same_v<decltype(x3),decltype(x4)>) 检查类型是否相同，结果为真，编译通过。

------

4. 使用 static_cast 显式转换

cpp

```cpp
X x5(static_cast<void(*)()>([]() { std::cout << "Same old.\n"; }));
static_assert(std::is_same_v<decltype(x4), decltype(x5)>);
```

- 这里通过 static_cast<void(*)()> 显式将 lambda 转换为函数指针类型 void(*)()。
- **关键点**：与单目 + 操作符类似，static_cast 强制 lambda 转换为函数指针类型。只要 lambda 没有捕获变量（这里没有），这种转换是合法的。
- 因此：
  - x5 的 Callback 类型是 void(*)()。
  - x4 的 Callback 类型也是 void(*)()（由前面的 + 转换而来）。
- decltype(x4) 和 decltype(x5) 都是 X<void(*)()>，类型相同。
- static_assert(std::is_same_v<decltype(x4), decltype(x5)>) 检查类型是否相同，结果为真，编译通过。

------

## 总结:

1. **Lambda 的独特性**：
   - 未经转换的 lambda（如 x1 和 x2）生成唯一的闭包类型，因此 X<闭包类型1> 和 X<闭包类型2> 是不同类型。
2. **函数指针转换**：
   - 使用 + 操作符（如 x3 和 x4）或 static_cast（如 x5）将 lambda 转换为函数指针类型 void(*)()。
   - 转换后的类型是固定的函数指针类型，不再依赖 lambda 的具体实现，因此 X<void(*)()> 对于所有无捕获的 lambda 都是相同的类型。
3. **static_assert 的作用**：
   - 验证了未经转换的 lambda 类型不同（x1 vs x2），而转换后的类型相同（x3 vs x4，x4 vs x5）。

------

输出解释

这段代码只是类型检查，不会产生运行时输出。如果编译通过，说明所有 static_assert 都成立。如果想实际调用这些 lambda，可以这样做：

cpp

```cpp
x1.call(); // 输出 "Hello World!"
x2.call(); // 输出 "Hello Universe!"
x3.call(); // 输出 "Hello World!"
x4.call(); // 输出 "Hello Universe!"
x5.call(); // 输出 "Same old."
```

希望这解释清楚了！如果有进一步疑问，欢迎提问！