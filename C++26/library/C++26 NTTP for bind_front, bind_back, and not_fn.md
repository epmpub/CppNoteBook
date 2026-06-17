这是 C++26 的一个小而实用的改进，来源于提案 **P2714R1: NTTP for `bind_front`, `bind_back`, and `not_fn`**。

它解决的问题是：

> 当目标可调用对象（callable）在编译期已知时，避免额外存储函数对象，减少类型大小，提高优化机会，并支持更多 constexpr 场景。

------

## C++23 之前

以 `std::bind_front` 为例：

```cpp
#include <functional>

int add(int a, int b)
{
    return a + b;
}

auto f = std::bind_front(add, 10);
```

实际上 `bind_front` 会生成类似：

```cpp
struct closure {
    int (*func)(int,int);
    int bound_arg;

    int operator()(int x) const {
        return func(bound_arg, x);
    }
};
```

对象内部需要保存：

```cpp
func = &add
bound_arg = 10
```

因此：

```cpp
sizeof(f)
```

至少包含：

```cpp
pointer + bound arguments
```

------

## C++26：NTTP Callable

新增重载：

```cpp
template<auto f, class... Args>
constexpr auto bind_front(Args&&... args);
```

这里的 callable 变成了 NTTP（Non-Type Template Parameter）。

可以写：

```cpp
auto f = std::bind_front<add>(10);
```

而不是：

```cpp
auto f = std::bind_front(add, 10);
```

------

## 对象大小变化

旧版本：

```cpp
auto f1 = std::bind_front(add, 10);
```

等价于：

```cpp
{
    int (*func)(int,int);
    int value;
}
```

可能是：

```cpp
sizeof(f1) == 16
```

（64位机器）

------

新版本：

```cpp
auto f2 = std::bind_front<add>(10);
```

等价于：

```cpp
{
    int value;
}
```

因为：

```cpp
add
```

已经成为模板参数：

```cpp
template<auto F>
```

无需存储。

可能：

```cpp
sizeof(f2) == 4
```

------

## bind_front 示例

### C++23

```cpp
int add(int a, int b)
{
    return a + b;
}

auto plus10 = std::bind_front(add, 10);

std::cout << plus10(5);
```

输出：

```text
15
```

------

### C++26

```cpp
auto plus10 = std::bind_front<add>(10);

std::cout << plus10(5);
```

同样输出：

```text
15
```

但对象更小。

------

## bind_back

同理：

```cpp
int sub(int a, int b)
{
    return a - b;
}
```

C++23：

```cpp
auto minus5 = std::bind_back(sub, 5);

minus5(20);
```

等价：

```cpp
sub(20,5)
```

------

C++26：

```cpp
auto minus5 = std::bind_back<sub>(5);
```

无需存储函数指针。

------

## not_fn

### C++23

```cpp
bool is_even(int x)
{
    return x % 2 == 0;
}

auto is_odd = std::not_fn(is_even);
```

内部需要保存：

```cpp
&is_even
```

------

### C++26

```cpp
auto is_odd = std::not_fn<is_even>();
```

调用：

```cpp
is_odd(3);
```

等价：

```cpp
!is_even(3)
```

函数地址不再需要存储。

------

## constexpr 改进

NTTP 的一个重要价值是增强编译期求值。

例如：

```cpp
constexpr int add(int a, int b)
{
    return a + b;
}

constexpr auto plus10 =
    std::bind_front<add>(10);

static_assert(plus10(20) == 30);
```

实现更容易做到完全 constexpr。

------

## 成员函数

对于成员函数也适用：

```cpp
struct X {
    int add(int x) const {
        return x + 100;
    }
};
```

C++26：

```cpp
auto f = std::bind_front<&X::add>(X{});

std::cout << f(1);
```

输出：

```text
101
```

因为：

```cpp
&X::add
```

也是合法 NTTP。

------

## Lambda 呢？

捕获为空的 lambda 可以。

例如：

```cpp
constexpr auto twice =
    [](int x)
    {
        return x * 2;
    };

auto f = std::bind_front<twice>();
```

因为无状态 lambda 是结构化类型（structural type），可以作为 NTTP。

------

带捕获的不行：

```cpp
int n = 10;

auto lam =
    [n](int x)
    {
        return x + n;
    };

std::bind_front<lam>(); // 错误
```

因为：

```cpp
lam
```

不是结构化类型。

------

## 为什么需要这个特性？

主要收益有四个：

### 1. 更小的闭包对象

以前：

```cpp
{
    callable;
    bound_args...
}
```

现在：

```cpp
{
    bound_args...
}
```

------

### 2. 更好的优化

编译器完全知道目标函数：

```cpp
bind_front<add>
```

可以直接：

- inline
- constant propagation
- dead code elimination

------

### 3. 更强 constexpr

更容易在编译期执行。

------

### 4. 与 ranges 风格一致

类似：

```cpp
std::views::transform<&foo>
```

这种“把行为放到模板参数里”的现代 C++ 风格。

------

## 一个完整示例

```cpp
#include <functional>
#include <print>

constexpr int add(int a, int b)
{
    return a + b;
}

constexpr bool is_even(int x)
{
    return x % 2 == 0;
}

int main()
{
    auto plus10 = std::bind_front<add>(10);

    std::println("{}", plus10(5));

    auto is_odd = std::not_fn<is_even>();

    std::println("{}", is_odd(3));

    static_assert(
        std::bind_front<add>(10)(20) == 30
    );
}
```

输出：

```text
15
true
```

------

从设计角度看，这项改动与 C++20/23 大量引入的 **NTTP + structural type** 思路一致：

```cpp
std::bind_front<func>()
std::bind_back<func>()
std::not_fn<func>()
```

把“调用目标”从运行期状态（函数指针成员）提升为编译期类型信息（模板参数），从而获得零存储开销、更好的优化和更强的 constexpr 能力。