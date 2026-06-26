## C++23 对 lambda 的增强

## 1. `static operator()`（静态调用运算符）

C++23 允许 lambda 的 `operator()` 是静态成员。

以前：

```cpp
auto f = []() {
    return 42;
};
```

本质类似：

```cpp
struct Closure {
    int operator()() const {
        return 42;
    }
};
```

C++23：

```cpp
auto f = [] static {
    return 42;
};
```

等价更接近：

```cpp
struct Closure {
    static int operator() {
        return 42;
    }
};
```

作用：

- 没有 `this`
- 不依赖闭包对象
- 更容易优化
- 更接近纯函数

尤其适合：

- policy object
- compile-time callable
- ranges predicate
- constexpr functional object

例如：

```cpp
auto add = [] static (int a, int b) {
    return a + b;
};
```

------

## 2. 显式对象参数（Explicit Object Parameter）

这是 C++23 lambda 最大增强之一。

以前 lambda 的 `this` 是隐式的：

```cpp
auto f = [=]() {
    return x;
};
```

C++23：

```cpp
auto f = [](this auto&& self, int x) {
    return self.value + x;
};
```

这是从 “成员函数模型” 借鉴来的。

本质：

```cpp
obj.f(a)
```

变成：

```cpp
f(obj, a)
```

意义非常大：

------

### (1) 支持递归 lambda

以前：

```cpp
auto fib = [&](int n) {
    return n <= 1 ? n : fib(n-1) + fib(n-2);
};
```

这是非法的，因为 `fib` 尚未完整定义。

旧方案：

```cpp
std::function<int(int)> fib;
fib = [&](int n) {
    return n <= 1 ? n : fib(n-1) + fib(n-2);
};
```

有性能损耗。

C++23：

```cpp
auto fib = [](this auto self, int n) {
    return n <= 1 ? n : self(n-1) + self(n-2);
};
```

无需：

- `std::function`
- 类型擦除
- 堆分配

这是非常重要的提升。

------

### (2) 更像真正成员函数

可以实现 CRTP 风格行为：

```cpp
auto printer = [](this auto&& self) {
    self.log();
};
```

lambda 不再只是“函数”。

而是：

- 可组合对象
- mixin
- strategy
- local customization point

------

## 3. lambda 可以有属性（attributes）增强

例如：

```cpp
auto f = [] [[nodiscard]] (int x) {
    return x * 2;
};
```

增强了 API 约束能力。

------

## 4. `constexpr` / `consteval` 更成熟

实际上从 C++20 开始已经大量增强。

C++23 继续改善 constexpr 环境。

例如：

```cpp
constexpr auto sq = [](int x) {
    return x * x;
};

static_assert(sq(5) == 25);
```

现代 lambda 已经逐渐成为：

- 编译期 DSL
- TMP 替代方案
- 元编程工具

------

# C++26 对 lambda 的趋势

截至目前（2026）：

C++26 对 lambda 没有像 C++23 那种“革命性语法变化”，但在几个方向继续强化。

重点是：

------

# 1. 更强 constexpr / 编译期执行能力

C++26 核心趋势：

- constexpr heap
- constexpr containers
- reflection
- compile-time execution

lambda 会自然受益。

例如未来：

```cpp
constexpr auto transform = [](auto r) {
    ...
};
```

可能在编译期直接运行复杂 ranges pipeline。

------

# 2. 与 Reflection 结合

C++26 Reflection（反射）推进后：

lambda 会成为 compile-time meta algorithm 的核心载体。

类似：

```cpp
template for (constexpr auto m : members_of(^^T)) {
    ...
}
```

内部通常会大量结合 lambda。

因为 lambda 是：

- 局部
- 可推导
- 泛型
- constexpr-friendly

非常适合作为元编程 callback。

------

# 3. 更深入融合 Ranges / Sender-Receiver

现代 C++ 越来越函数式：

```cpp
views::transform(...)
views::filter(...)
```

lambda 已经是 ranges 的核心语法。

C++26 的 executors / sender-receiver 会进一步强化：

```cpp
then([](auto v) {
    return process(v);
})
```

lambda 将成为：

- async continuation
- coroutine callback
- execution graph node

------

# 4. 泛型 lambda 更接近真正函数模板

现在：

```cpp
auto f = []<typename T>(T x) {
    return x;
};
```

实际上已经非常接近：

```cpp
template<typename T>
T f(T x);
```

未来标准趋势：

- 更好约束
- 更好推导
- 更好 overload
- 更好 reflection

lambda 正在逐渐“模板化”。

------

# 一个非常关键的趋势

现代 C++ 中：

> lambda 不再只是匿名函数。

而是：

- 局部类
- compile-time object
- policy type
- customization point
- continuation
- functional combinator

甚至某种程度：

> lambda 正在替代很多“小型 class + operator()”。

------

例如以前：

```cpp
struct Add {
    int operator()(int a, int b) const {
        return a + b;
    }
};
```

现代：

```cpp
constexpr auto add = [](int a, int b) {
    return a + b;
};
```

lambda 已经成为：

- STL
- Ranges
- Coroutine
- Executors
- Metaprogramming

中的基础构件。

------

一个简单总结：

| 版本  | 增强重点                                      |
| ----- | --------------------------------------------- |
| C++11 | 基础 lambda                                   |
| C++14 | 泛型 lambda                                   |
| C++17 | constexpr 增强                                |
| C++20 | template lambda / concepts                    |
| C++23 | recursive lambda / explicit object parameter  |
| C++26 | compile-time + reflection + async integration |

C++ 的 lambda 正在从“语法糖”演化成“核心抽象机制”。