#### trivial types & non-trivial types

C++ 中的 **trivial type（平凡类型）** 是一个非常重要的概念，它决定了对象是否可以通过简单的内存操作（如 `memcpy`、静态存储区零初始化等）来创建、复制和销毁。

不过要注意，标准中有几个容易混淆的概念：

```text
Trivial type
Trivially copyable type
Standard-layout type
POD type（C++20已废弃）
```

很多人实际上想问的是 **Trivially Copyable**，而不是 **Trivial**。

------

## 1. Trivial Type 的正式定义

一个类型 `T` 是 **trivial type**，当且仅当：

```text
1. T 是标量类型（scalar type）
   或

2. T 是 trivial class
```

其中标量类型包括：

```cpp
int
double
char
bool
enum
pointer
nullptr_t
```

例如：

```cpp
static_assert(std::is_trivial_v<int>);
static_assert(std::is_trivial_v<double>);
static_assert(std::is_trivial_v<int*>);
```

全部成立。

------

## 2. 什么是 trivial class

一个类要成为 trivial class，需要同时满足：

### (1) 至少有一个可用的默认构造函数

例如：

```cpp
struct A
{
    int x;
};
```

编译器隐式生成：

```cpp
A();
```

------

### (2) 默认构造函数是 trivial 的

不能自己写：

```cpp
struct B
{
    B() {}
};
```

因为：

```cpp
B()
```

已经变成 user-provided constructor。

结果：

```cpp
static_assert(!std::is_trivial_v<B>);
```

------

正确：

```cpp
struct C
{
    C() = default;
};
```

仍然是 trivial。

------

### (3) 所有拷贝/移动构造函数都是 trivial

例如：

```cpp
struct D
{
    int x;
};
```

编译器生成：

```cpp
D(const D&);
D(D&&);
```

都是 trivial。

------

但：

```cpp
struct E
{
    E(const E&) {}
};
```

则不再 trivial。

------

### (4) 所有拷贝/移动赋值都是 trivial

例如：

```cpp
struct F
{
    int x;
};
```

满足。

------

但：

```cpp
struct G
{
    G& operator=(const G&)
    {
        return *this;
    }
};
```

不满足。

------

### (5) 析构函数是 trivial

例如：

```cpp
struct H
{
    int x;
};
```

析构函数隐式生成：

```cpp
~H();
```

是 trivial。

------

但：

```cpp
struct I
{
    ~I() {}
};
```

不是 trivial。

------

### (6) 所有成员和基类也是 trivial

例如：

```cpp
struct Member
{
    ~Member() {}
};

struct J
{
    Member m;
};
```

则：

```cpp
J
```

也不 trivial。

------

## 3. 最简单的 trivial class

```cpp
struct Point
{
    int x;
    int y;
};
```

满足：

```cpp
static_assert(std::is_trivial_v<Point>);
```

成立。

内存布局：

```text
+-----+-----+
|  x  |  y  |
+-----+-----+
```

编译器无需生成任何特殊代码。

------

## 4. 失去 trivial 的常见情况

### 自定义构造函数

```cpp
struct A
{
    A() {}
};
```

结果：

```cpp
std::is_trivial_v<A> == false
```

------

### 自定义析构函数

```cpp
struct B
{
    ~B() {}
};
```

结果：

```cpp
false
```

------

### 自定义拷贝构造

```cpp
struct C
{
    C(const C&) {}
};
```

结果：

```cpp
false
```

------

### 虚函数

```cpp
struct D
{
    virtual void f() {}
};
```

结果：

```cpp
false
```

因为有：

```text
vptr
```

以及非平凡构造语义。

------

### 虚继承

```cpp
struct Base {};

struct E : virtual Base {};
```

结果：

```cpp
false
```

------

## 5. trivial ≠ trivially copyable

很多人混淆这两个概念。

例如：

```cpp
struct A
{
    A() {}
    int x;
};
```

不是 trivial：

```cpp
static_assert(!std::is_trivial_v<A>);
```

但却是：

```cpp
static_assert(std::is_trivially_copyable_v<A>);
```

成立。

因为：

```cpp
A
```

仍然可以安全 `memcpy`。

只是构造不再是 trivial。

------

## 6. C++26 示例

```cpp
#include <type_traits>

struct S1
{
    int x;
};

struct S2
{
    S2() {}
    int x;
};

struct S3
{
    ~S3() {}
};

int main()
{
    static_assert(std::is_trivial_v<int>);
    static_assert(std::is_trivial_v<S1>);

    static_assert(!std::is_trivial_v<S2>);
    static_assert(!std::is_trivial_v<S3>);
}
```

------

## 7. C++26 的变化

需要特别注意：

### `std::is_trivial`

已经被 C++26 标记为废弃：

```cpp
[[deprecated]]
std::is_trivial
```

原因是：

```text
trivial
```

这个概念在实际工程中的用途越来越少。

标准委员会认为：

```text
trivially copyable
trivially constructible
trivially destructible
```

这些更细粒度的 trait 更有价值。

因此未来更推荐：

```cpp
std::is_trivially_copyable_v<T>

std::is_trivially_default_constructible_v<T>

std::is_trivially_destructible_v<T>
```

而不是：

```cpp
std::is_trivial_v<T>
```

------

总结一下：

一个类型是 **trivial type** 的核心要求是：

```text
- 默认构造是 trivial
- 拷贝构造是 trivial
- 移动构造是 trivial
- 拷贝赋值是 trivial
- 移动赋值是 trivial
- 析构函数是 trivial
- 所有基类和成员也满足上述条件
- 没有虚函数/虚基类等复杂对象语义
```

典型例子：

```cpp
struct Point
{
    int x;
    int y;
};
```

这是最经典的 trivial type。只要你开始手写构造函数、析构函数、拷贝操作或引入虚函数，通常就会失去 trivial 性质。