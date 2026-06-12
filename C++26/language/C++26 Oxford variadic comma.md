## C++26 Oxford variadic comma

在 C++26 中，**[P3176R1 提案（The Oxford variadic comma）](https://github.com/cplusplus/papers/issues/1839)** 引入了一项关于语言规范的清理工作：**正式弃用（Deprecate）在声明不带命名参数的 C 风格变长参数函数（Variadic Functions）时，省略省略号（`...`）前面的逗号的行为**。 [1] 

这个提案的名字借用了英语语法中的“**牛津逗号（Oxford Comma）**”（即在一串并列词语的最后两个词之间，在“and”或“or”前面加的那个逗号）。在 C++26 里，它代表着**在变长参数的 `...` 之前必须加一个“逗号”**。 [1] 

------

## 历史背景：可以被省略的逗号

在 C++23 及更早的标准中（实际上是从 C++98 开始为了向后兼容），声明一个类似 C 风格 `printf` 的变长参数函数时，`...` 前面的逗号是**可选**的。 [2] 

```cpp
// 传统的标准写法（带逗号）
void print_values(int count, ...);

// 鲜为人知的 C++ 特有遗留写法（省略了逗号！）
// 在 C++23 中是完全合法的
void print_values_legacy(int count ...); 
```

这种不带逗号的写法（`int count ...`）在现代 C++ 源码中极其罕见，甚至是很多资深 C++ 开发者都不知道的冷知识。 [2] 

------

## C++26 的彻底改变

为了消除语法的二义性、提升代码可读性并为未来的新特性铺路，C++26 通过 P3176R1 宣布：**省略逗号的变长参数声明已被正式弃用（Deprecated）**。 [1, 3] 

## C++26 警告/弃用代码

```cpp
// C++26 编译时会触发 Deprecation 警告（在未来标准中可能直接变为错误）
void process(int 标识符 ...); 
```

## C++26 推荐代码

```cpp
// 必须明确加上“牛津逗号”
void process(int 标识符, ...); 
```

------

## 为什么要做出这项改动？

引入“牛津变长参数逗号”强制规范，主要有以下三个核心考量：

## 1. 消除词法解析的二义性与混乱

在没有逗号的情况下，`...` 紧跟在类型或标识符后面，极易与 C++11 引入的**可变参数模板（Parameter Packs / Variadic Templates）**的语法发生视觉和解析上的混淆。

- `void foo(Args... args);` -> 现代 C++ 模板参数包展开。
- `void foo(int x...);` -> C 风格老旧变长参数（没有任何类型安全保护）。
  强制加上逗号后，`void foo(int x, ...);` 可以让阅读代码的人和编译器一眼看出这是危险的、非类型安全的 C 风格变长参数。

## 2. 与 C 语言标准对齐（C Compatibility）

在 C 语言中，`...` 前面的逗号**从来都是强制要求的**（例如 C 标准里的 `int printf(const char *format, ...);`）。C++ 过去允许省略逗号其实是其自身发展史上的一个特有“异项”。C++26 移除这个特例，使两者在 C 风格变长参数的语法定义上达成了完全一致。 [1, 3] 

## 3. 为未来的语言新特性扫清语法障碍

C++ 委员会正积极为未来的标准（如 C++26/C++29）规划更多关于模式匹配（Pattern Matching）和参数包扩展（Pack Extensions）的宏大特性。清理掉这种带有二义性的边缘老旧语法，可以防止它污染后续新特性的关键字设计和编译器的语法解析器。

## 总结

对于 **99.9% 的现代 C++ 开发者**来说，这个改动**没有任何负面影响**，因为几乎没人会写 `int count ...` 这样不加逗号的代码。现代 C++ 中我们也更倾向于使用可变参数模板（`typename... Args`）或 C++23 的 `std::print`，而不是传统的 C 风格变长参数。这一波清理，纯粹是 C++ 标准库的一场“语法强迫症式”的规范化洗礼。 [2] 

你是否在老旧的 C++ 遗留代码库中见过这种不带逗号的怪异写法？我们可以聊聊如何借助现代 C++ 模板将这些老旧的变长参数函数重构得更加类型安全！

[1] [https://www.sandordargo.com](https://www.sandordargo.com/blog/2026/03/11/cpp26-oxford-variadic-comma)

[2] [https://news.ycombinator.com](https://news.ycombinator.com/item?id=47332593)

[3] [https://stackoverflow.com](https://stackoverflow.com/questions/35708014/comma-omitted-in-variadic-function-declaration-in-c)