`static operator()` 是 C++23 的一个新特性（提案 P1169R4），允许把函数调用运算符声明为 `static`。

在 C++23 之前：

```cpp
struct Foo {
    void operator()() const {
        std::cout << "call\n";
    }
};
```

`operator()` 必须是非静态成员函数，因此总会有一个隐含的 `this` 指针。

------

## C++23 写法

```cpp
struct Foo {
    static void operator()() {
        std::cout << "call\n";
    }
};

int main()
{
    Foo{}();
}
```

输出：

```text
call
```

------

## 为什么需要它

观察一个无状态仿函数：

```cpp
struct Plus {
    int operator()(int a, int b) const {
        return a + b;
    }
};
```

实际上：

- 不访问成员变量
- 不访问 `this`
- 完全是一个普通函数

但编译器仍然必须把它当作成员函数：

```cpp
plus.operator()(1, 2);
```

等价于：

```cpp
Plus::operator()(&plus, 1, 2);
```

其中隐藏传递了：

```cpp
this
```

而：

```cpp
struct Plus {
    static int operator()(int a, int b) {
        return a + b;
    }
};
```

则没有 `this`。

------

## 示例

### 传统写法

```cpp
struct Add {
    int operator()(int a, int b) const {
        return a + b;
    }
};
```

编译器概念上生成：

```cpp
int Add::operator()(Add const* this,
                    int a,
                    int b);
```

------

### C++23

```cpp
struct Add {
    static int operator()(int a, int b) {
        return a + b;
    }
};
```

概念上变成：

```cpp
static int Add::operator()(int a,
                           int b);
```

没有 `this` 参数。

------

## 调用方式

虽然是静态成员函数：

```cpp
struct Add {
    static int operator()(int a, int b) {
        return a + b;
    }
};
```

仍然可以：

```cpp
Add add;

std::cout << add(1, 2);
```

输出：

```text
3
```

编译器会把：

```cpp
add(1, 2)
```

转换为：

```cpp
Add::operator()(1, 2);
```

------

## 也可以直接类名调用

```cpp
std::cout << Add::operator()(1, 2);
```

输出：

```text
3
```

这一点是普通 `operator()` 做不到的。

------

## 与 Lambda 的关系

无状态 Lambda：

```cpp
auto f = [](int a, int b) {
    return a + b;
};
```

其闭包类型大致类似：

```cpp
class Lambda {
public:
    int operator()(int a, int b) const {
        return a + b;
    }
};
```

实际上并不需要 `this`。

C++23 允许实现成：

```cpp
class Lambda {
public:
    static int operator()(int a, int b) {
        return a + b;
    }
};
```

从而减少额外开销。

因此这是为：

- Lambda
- Ranges
- Function Objects
- 标准库谓词对象

提供更好的实现基础。

------

## 限制

### 不能访问非静态成员

```cpp
struct Foo {
    int x = 42;

    static int operator()() {
        return x;      // ERROR
    }
};
```

因为没有：

```cpp
this
```

------

### 只能访问静态成员

```cpp
struct Foo {
    static inline int value = 42;

    static int operator()() {
        return value;
    }
};
```

合法。

------

## 与 static operator[] 一起引入

C++23 不仅允许：

```cpp
static operator()
```

还允许：

```cpp
static operator[]
```

例如：

```cpp
struct Matrix {
    static int operator[](int i) {
        return i * i;
    }
};
```

------

## 实际价值

标准库中有大量无状态函数对象：

例如：

std::less

```cpp
struct less {
    template<class T>
    constexpr bool operator()(T const& a,
                              T const& b) const
    {
        return a < b;
    }
};
```

理论上可写为：

```cpp
struct less {
    template<class T>
    static constexpr bool operator()(T const& a,
                                     T const& b)
    {
        return a < b;
    }
};
```

更符合其本质。

------

## 一个完整示例

```cpp
#include <iostream>

struct Printer {
    static void operator()(const char* msg)
    {
        std::cout << msg << '\n';
    }
};

int main()
{
    Printer p;

    p("hello");

    Printer::operator()("world");
}
```

输出：

```text
hello
world
```

------

总结：

`static operator()` 是 C++23 新增能力，它允许函数调用运算符不依赖对象实例。

```cpp
struct Add {
    static int operator()(int a, int b) {
        return a + b;
    }
};
```

特点：

- 没有 `this` 指针
- 不能访问非静态成员
- 仍可通过 `obj(...)` 语法调用
- 也可通过 `Type::operator()(...)` 调用
- 特别适合无状态仿函数和无捕获 Lambda 的实现优化

它本质上让“仿函数”可以更接近“普通函数”，同时保留运算符调用语法。