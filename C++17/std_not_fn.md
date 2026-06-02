`std::not_fn` 是 C++17 引入的函数适配器（Function Adapter），定义在：

```cpp
#include <functional>
```

作用非常简单：

> **把一个可调用对象（函数、Lambda、仿函数、成员函数指针等）的返回结果取反。**

即：

```cpp
f(args...)
```

变成：

```cpp
!f(args...)
```

------

## 为什么需要它？

在 C++11/14 时代，经常会遇到这样的代码：

```cpp
std::find_if(v.begin(), v.end(),
    [](int x) {
        return !is_even(x);
    });
```

为了取反一个谓词（Predicate），不得不重新写一个 lambda。

如果谓词很复杂：

```cpp
auto pred = [](const Person& p) {
    return p.age > 18 && p.salary > 10000;
};
```

想取反：

```cpp
auto not_pred = [](const Person& p) {
    return !(p.age > 18 && p.salary > 10000);
};
```

实际上只是：

```cpp
!pred(...)
```

标准库认为这种需求太常见了，于是提供：

```cpp
std::not_fn(pred)
```

------

# 最简单例子

```cpp
#include <functional>
#include <iostream>

bool is_even(int x)
{
    return x % 2 == 0;
}

int main()
{
    auto is_odd = std::not_fn(is_even);

    std::cout << is_odd(3) << '\n'; // true
    std::cout << is_odd(4) << '\n'; // false
}
```

等价于：

```cpp
auto is_odd = [](auto&&... args)
{
    return !is_even(
        std::forward<decltype(args)>(args)...);
};
```

------

# 在算法中的价值

例如：

```cpp
std::vector<int> v{1,2,3,4,5,6};
```

找第一个奇数：

### 传统写法

```cpp
auto it =
    std::find_if(
        v.begin(),
        v.end(),
        [](int x)
        {
            return !is_even(x);
        });
```

------

### 使用 not_fn

```cpp
auto it =
    std::find_if(
        v.begin(),
        v.end(),
        std::not_fn(is_even));
```

更直接表达：

```text
查找非 is_even 的元素
```

------

# 支持 Lambda

```cpp
auto adult = [](const Person& p)
{
    return p.age >= 18;
};

auto child = std::not_fn(adult);
```

等价于：

```cpp
auto child = [](const Person& p)
{
    return !adult(p);
};
```

------

# 支持成员函数

这是 `not_fn` 的一个重要优势。

假设：

```cpp
struct Person
{
    bool isAdult() const
    {
        return age >= 18;
    }

    int age;
};
```

可以：

```cpp
auto child =
    std::not_fn(&Person::isAdult);
```

使用：

```cpp
Person p{10};

std::cout << child(p);
```

输出：

```text
true
```

------

# 与 binders 的关系

在 C++98 时代有：

```cpp
std::not1
std::not2
```

例如：

```cpp
std::not1(pred)
```

用于一元谓词。

```cpp
std::not2(pred)
```

用于二元谓词。

但有很多限制：

- 必须继承特定基类
- 必须定义 typedef
- 只能支持固定参数个数

例如：

```cpp
std::unary_function
std::binary_function
```

这些设计后来被废弃。

------

C++17 的：

```cpp
std::not_fn
```

本质是现代版：

```cpp
std::not1
std::not2
```

统一替代品。

------

# 一个近似实现

可以理解为：

```cpp
template<class F>
auto not_fn(F&& f)
{
    return [f = std::forward<F>(f)]
           (auto&&... args)
    {
        return !std::invoke(
            f,
            std::forward<decltype(args)>(args)...);
    };
}
```

关键点：

```cpp
std::invoke(...)
```

因此它支持：

- 普通函数
- Lambda
- 仿函数
- 成员函数指针
- 成员变量指针

所有 Callable。

------

# C++20 Ranges 中为什么不常见了

例如：

```cpp
std::ranges::copy_if(
    v,
    out,
    std::not_fn(is_even));
```

仍然合法。

但很多人更喜欢：

```cpp
v
| std::views::filter(
      [](int x){ return !is_even(x); });
```

或者：

```cpp
v
| std::views::filter(
      std::not_fn(is_even));
```

因此在 Ranges 时代，`not_fn` 更多是一种组合工具。

------

# 本质总结

`std::not_fn` 解决的问题其实很单纯：

假设已有谓词：

```cpp
bool pred(T);
```

想得到：

```cpp
bool not_pred(T)
{
    return !pred(T);
}
```

以前必须手写：

```cpp
[](auto&&... args)
{
    return !pred(args...);
}
```

C++17 提供：

```cpp
auto not_pred = std::not_fn(pred);
```

让“谓词取反”成为一个标准化、泛型化、支持 `std::invoke` 的函数适配器。

它属于 STL 函数式编程工具链的一部分，和：

```cpp
std::bind
std::mem_fn
std::invoke
std::reference_wrapper
```

属于同一类“Callable Adapter”。