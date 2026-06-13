最能体现 `explicit(bool)` 价值的例子，是实现一个类似 `std::optional<T>` 的包装器。

假设要求：

- 如果 `U` 可以隐式转换为 `T`，那么允许：

```cpp
Wrapper<int> w = 42;
```

- 如果 `U` 不能隐式转换为 `T`，则必须显式构造：

```cpp
Wrapper<std::string> w("hello");
```

而不允许：

```cpp
Wrapper<std::string> w = "hello";
```

------

### C++17 的写法

需要两套构造函数：

```cpp
#include <type_traits>

template<typename T>
class Wrapper {
public:
    template<typename U,
             std::enable_if_t<
                 std::is_convertible_v<U,T>, int> = 0>
    Wrapper(U&& u)
        : value(std::forward<U>(u))
    {}

    template<typename U,
             std::enable_if_t<
                 !std::is_convertible_v<U,T>, int> = 0>
    explicit Wrapper(U&& u)
        : value(std::forward<U>(u))
    {}

private:
    T value;
};
```

这里逻辑完全一样，只是为了控制 `explicit`，不得不写两份代码。

------

### C++20 的写法

```cpp
#include <type_traits>

template<typename T>
class Wrapper {
public:

    template<typename U>
    explicit(!std::is_convertible_v<U,T>)
    Wrapper(U&& u)
        : value(std::forward<U>(u))
    {}

private:
    T value;
};
```

只保留一份构造函数。

------

看看实例化后会发生什么。

------

## 情况1：允许隐式转换

```cpp
Wrapper<int> a = 123;
```

此时：

```cpp
U = int
T = int
```

计算：

```cpp
std::is_convertible_v<int,int>
```

结果：

```cpp
true
```

因此：

```cpp
explicit(!true)
```

即：

```cpp
explicit(false)
```

等价于：

```cpp
Wrapper(int);
```

允许：

```cpp
Wrapper<int> a = 123;
```

------

## 情况2：禁止隐式转换

```cpp
struct UserId {
    explicit UserId(int) {}
};

Wrapper<UserId> x = 123;
```

此时：

```cpp
U = int
T = UserId
```

计算：

```cpp
std::is_convertible_v<int, UserId>
```

结果：

```cpp
false
```

因为：

```cpp
UserId(int)
```

本身就是 explicit。

于是：

```cpp
explicit(!false)
```

变成：

```cpp
explicit(true)
```

等价于：

```cpp
explicit Wrapper(int);
```

因此：

```cpp
Wrapper<UserId> x = 123;
```

编译失败。

只能：

```cpp
Wrapper<UserId> x(123);
```

------

### 一个更直观的例子

假设你设计一个数据库主键类型：

```cpp
struct UserId {
    explicit UserId(int id)
        : id(id)
    {}

    int id;
};
```

你不希望程序员误写：

```cpp
UserId id = 100;
```

因为这是普通整数到业务类型的隐式转换。

必须写：

```cpp
UserId id(100);
```

这样代码更安全。

然后你再写：

```cpp
template<typename T>
struct Box {

    template<typename U>
    explicit(!std::is_convertible_v<U,T>)
    Box(U&& value)
        : value(std::forward<U>(value))
    {}

    T value;
};
```

此时：

```cpp
Box<int> a = 100;      // OK
Box<UserId> b = 100;   // ERROR
Box<UserId> c(100);    // OK
```

`Box` 自动继承了 `UserId` 的显式构造语义。

------

这正是标准库中的 `std::pair`、`std::tuple`、`std::optional`、`std::expected` 等类型使用 `explicit(bool)` 的原因：

它们希望：

> “如果底层类型允许隐式转换，我也允许；如果底层类型要求显式构造，我也要求。”

而在 C++20 之前，这通常需要两套重载；C++20 以后只需：

```cpp
explicit(!std::is_convertible_v<U,T>)
```

一行就能表达整个规则。