consexpr编译时问题：

```tex
a constexpr variable must have a literal type or a reference type

is not literal because:
 
does not have ‘constexpr’ destructor
```



当编译器报错：

```text
constexpr variable must have a literal type
```

意思是：

```text
constexpr 变量的类型必须是 Literal Type（字面量类型）
```

因为 `constexpr` 对象要求能够在编译期完全构造出来，而编译器只有对某些类型才能做到这一点。

例如：

```cpp
constexpr int x = 42;
```

这里 `int` 就是 Literal Type。

而：

```cpp
std::string s = "Hello";

constexpr std::string x = "Hello"; // C++17错误
```

在 C++17 中会报类似错误，因为当时 `std::string` 不是 Literal Type。

------

## Literal Type 的本质

可以简单理解为：

```text
编译器能够在编译期创建、复制、销毁该类型对象
```

这样的类型称为 Literal Type。

例如：

```cpp
constexpr int x = 42;
constexpr double pi = 3.14;
constexpr bool b = true;
```

都合法。

因为：

```cpp
int
double
bool
```

都是 Literal Type。

------

## 哪些类型天然是 Literal Type

### 标量类型（Scalar Types）

```cpp
int
char
double
bool
enum
pointer
nullptr_t
```

例如：

```cpp
constexpr int x = 1;
constexpr double y = 3.14;
constexpr int* p = nullptr;
```

全部合法。

------

### 引用

```cpp
constexpr int value = 42;
constexpr const int& ref = value;
```

合法。

------

## 类类型要满足什么条件

对于类：

```cpp
struct Point
{
    int x;
    int y;
};
```

这是 Literal Type：

```cpp
constexpr Point p{1,2};
```

合法。

------

### C++20之前

类必须满足：

- 析构函数是 trivial
- 所有成员都是 Literal Type
- 至少有一个 constexpr 构造函数

例如：

```cpp
struct Point
{
    int x;
    int y;

    constexpr Point(int a,int b)
        : x(a), y(b)
    {
    }
};
```

------

### C++20之后

规则放宽很多。

例如：

```cpp
struct Point
{
    int x;
    int y;

    constexpr Point(int a,int b)
        : x(a), y(b)
    {
    }

    constexpr ~Point() = default;
};
```

仍然是 Literal Type。

甚至很多标准库类型开始成为 Literal Type。

------

## 不是 Literal Type 的典型例子

### 虚函数

```cpp
struct Base
{
    virtual void f() {}
};
```

则：

```cpp
constexpr Base b{};
```

错误。

因为含有虚函数。

------

### 非 Literal 成员

```cpp
struct A
{
    std::mutex m;
};
```

则：

```cpp
constexpr A a{};
```

错误。

因为：

```cpp
std::mutex
```

不是 Literal Type。

------

## 为什么 std::string 曾经不行

C++17：

```cpp
constexpr std::string s = "hello";
```

错误。

因为：

```cpp
std::string
```

涉及动态内存分配。

而当时 constexpr 不允许。

------

C++20以后：

```cpp
constexpr std::string s{"hello"};
```

在支持的实现上可能合法。

因为：

- constexpr 动态分配被允许
- string 的很多成员变成 constexpr

所以：

```cpp
std::string
```

逐渐成为 Literal Type。

------

## 如何判断一个类型是不是 Literal Type

遗憾的是：

```cpp
std::is_literal_type
```

已经废弃：

```cpp
std::is_literal_type_v<T>
```

C++17 废弃，C++20 删除。

例如：

```cpp
static_assert(std::is_literal_type_v<int>);
```

现代标准已经不推荐。

------

## 现代判断方式

实际上直接测试：

```cpp
constexpr T obj{};
```

能否编译即可。

例如：

```cpp
struct Point
{
    int x;
    int y;
};

constexpr Point p{};
```

编译成功：

```text
Point 是 Literal Type
```

------

## 一个容易混淆的例子

```cpp
struct A
{
    int x;

    A(int n)
        : x(n)
    {
    }
};
```

这里：

```cpp
constexpr A a{42};
```

错误。

因为构造函数不是 constexpr。

改成：

```cpp
struct A
{
    int x;

    constexpr A(int n)
        : x(n)
    {
    }
};
```

则：

```cpp
constexpr A a{42};
```

合法。

------

## 与 Trivial Type 的关系

很多人把两者混为一谈。

```cpp
struct A
{
    int x;
    int y;
};
```

既是：

```text
Literal Type
Trivial Type
```

------

但：

```cpp
struct A
{
    int x;

    constexpr A(int n)
        : x(n)
    {
    }
};
```

则：

```text
Literal Type
```

成立，

而：

```text
Trivial Type
```

不成立（因为有用户定义构造函数）。

所以：

```text
Literal Type ≠ Trivial Type
```

Literal Type 的要求比 Trivial Type 更宽松。

------

从 C++20/C++23/C++26 的角度，可以把 Literal Type 理解成：

```text
一种允许出现在 constexpr 对象、
constexpr 函数、
consteval 函数中的类型。
```

只要编译器能够在编译期完成该类型对象的完整生命周期（构造、使用、销毁），它就属于 Literal Type。
