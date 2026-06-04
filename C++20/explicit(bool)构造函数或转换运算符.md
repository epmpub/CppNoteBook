`explicit(bool)` 是 C++20 引入的新特性（提案 P0892），允许你**有条件地将构造函数或转换运算符声明为 explicit**。

在 C++20 之前，你只能写：

```cpp
explicit Foo(int);   // 始终 explicit
```

或者：

```cpp
Foo(int);            // 始终允许隐式转换
```

不能根据模板参数或编译期条件决定。

------

## 基本语法

```cpp
explicit(condition)
Foo(...);
```

其中 `condition` 必须是一个编译期常量表达式。

等价于：

```cpp
if constexpr (condition)
    explicit;
else
    non-explicit;
```

只是这种效果由编译器实现。

------

## 最经典的例子：std::pair 和 std::tuple

C++17 以前标准库内部要写大量模板技巧：

```cpp
template<typename U1, typename U2>
pair(U1&&, U2&&);
```

需要判断：

```cpp
is_convertible_v<U1, T1>
is_convertible_v<U2, T2>
```

如果都能隐式转换：

```cpp
pair p = {1, 2};
```

允许。

否则：

```cpp
pair p = {obj1, obj2};
```

禁止。

------

C++20 后可以直接写：

```cpp
template<class U1, class U2>
explicit(
    !std::is_convertible_v<U1, T1> ||
    !std::is_convertible_v<U2, T2>
)
pair(U1&& x, U2&& y);
```

意思：

- 都可隐式转换 → 构造函数非 explicit
- 只要有一个不可隐式转换 → 构造函数 explicit

------

## 示例1

```cpp
#include <iostream>

struct Foo {
    explicit(false) Foo(int) {}
};

int main() {
    Foo f = 42;   // OK
}
```

因为：

```cpp
explicit(false)
```

等价于：

```cpp
Foo(int);
```

------

## 示例2

```cpp
struct Foo {
    explicit(true) Foo(int) {}
};

int main() {
    Foo f = 42;   // ERROR
}
```

等价于：

```cpp
explicit Foo(int);
```

必须：

```cpp
Foo f(42);
```

------

## 模板中的价值

假设我们实现一个智能包装器：

```cpp
template<typename T>
struct Wrapper {
    explicit(!std::is_integral_v<T>)
    Wrapper(T value)
        : value(value)
    {}

    T value;
};
```

------

### 整数类型

```cpp
Wrapper<int> a = 10;
```

允许：

```cpp
std::is_integral_v<int>
```

为 true，因此：

```cpp
explicit(false)
```

即允许隐式转换。

------

### 非整数类型

```cpp
Wrapper<std::string> s = "abc";
```

报错。

因为：

```cpp
explicit(true)
```

需要：

```cpp
Wrapper<std::string> s("abc");
```

------

## 转换运算符

同样适用于：

```cpp
operator bool()
```

例如：

```cpp
template<typename T>
struct SmartPtr {

    explicit(!std::is_pointer_v<T>)
    operator bool() const {
        return true;
    }
};
```

编译器会根据模板参数决定：

```cpp
operator bool()
```

是否允许隐式转换。

------

## 标准库中的实际用途

C++20 后大量标准库组件采用：

- `std::pair`
- `std::tuple`
- `std::optional`
- `std::expected`（C++23）
- `std::variant`
- `std::ranges` 相关类型

以前需要：

```cpp
enable_if
is_convertible
```

写两套构造函数：

```cpp
explicit Foo(...)
Foo(...)
```

现在一套即可：

```cpp
explicit(condition)
Foo(...)
```

------

## 一个完整例子

```cpp
#include <iostream>
#include <type_traits>

template<typename T>
struct Box {

    explicit(!std::is_arithmetic_v<T>)
    Box(T v)
        : value(v)
    {}

    T value;
};

int main() {

    Box<int> a = 10;     // OK

    Box<int> b(20);      // OK

    // Box<std::string> c = "abc"; // ERROR

    Box<std::string> d("abc");     // OK
}
```

这里：

```cpp
Box<int>
```

构造函数是：

```cpp
explicit(false)
```

允许隐式转换。

而：

```cpp
Box<std::string>
```

构造函数是：

```cpp
explicit(true)
```

禁止隐式转换。

------

从语言设计角度看，`explicit(bool)` 解决的是：

> **模板类或泛型库中，构造函数/转换运算符是否 explicit 需要依赖模板参数才能决定的问题。**

这是 C++20 对泛型编程的一项重要增强，使许多原本需要 SFINAE、`enable_if`、重载分发才能实现的逻辑，可以直接在声明处表达出来。