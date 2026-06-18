这是 C++26 的 **P2264R7: Make assert() macro user friendly for C and C++**，目标很简单：

> 让 `assert` 能够直接接受包含逗号的表达式，而不必再额外套一层括号。

这是一个长期困扰 C/C++ 开发者的可用性问题。

### C++23 之前的问题

`assert` 本质上是宏：

```cpp
#define assert(expr) ...
```

预处理器在展开宏时并不理解 C++ 语法，只认识逗号分隔参数。

例如：

```cpp
assert(std::is_same_v<int, int>);
```

看起来没问题。

但：

```cpp
assert(std::is_same_v<int, long>);
```

实际上预处理器看到的是：

```cpp
assert(
    std::is_same_v<int,
    long>
);
```

逗号会被解释成：

```text
两个宏参数
```

导致编译错误：

```text
macro "assert" passed 2 arguments
```

因此过去必须写：

```cpp
assert((std::is_same_v<int, long>));
```

多包一层括号。

------

类似问题还有：

```cpp
assert(std::pair<int,int>{}.first == 0);
assert(std::tuple<int,double>{}.index() == 0);
assert(std::is_convertible_v<A,B>);
```

都可能因为模板参数中的逗号而出问题。

开发者不得不记住：

```cpp
assert((expression));
```

而不是：

```cpp
assert(expression);
```

------

### C++26 的解决方案

标准把 `assert` 改成变参宏（variadic macro）：

原来类似：

```cpp
#define assert(expr)
```

变成：

```cpp
#define assert(...)
```

内部再把：

```cpp
__VA_ARGS__
```

作为一个整体表达式处理。

因此：

```cpp
assert(std::is_same_v<int, long>);
```

现在合法。

------

### 例子

C++23：

```cpp
assert(std::is_same_v<int,long>);
```

可能报错：

```text
macro passed 2 arguments
```

必须写：

```cpp
assert((std::is_same_v<int,long>));
```

------

C++26：

```cpp
assert(std::is_same_v<int,long>);
```

直接合法。

------

再看一个真实例子：

```cpp
#include <cassert>
#include <type_traits>

int main()
{
    assert(std::is_same_v<int,int>);
}
```

C++23：

```text
可能失败
```

C++26：

```text
正常编译
```

------

### 为什么以前不能这样改？

因为历史原因。

最早的 C89：

```cpp
assert(expr)
```

已经广泛实现。

后来：

```cpp
assert(x > 0)
```

这种用法遍布几十年代码库。

标准委员会一直担心修改：

```cpp
assert(expr)
```

的形式会破坏兼容性。

直到确认：

```cpp
#define assert(...)
```

对现有代码几乎完全兼容。

于是最终在 C23 和 C++26 一起采用。

------

### 与消息输出的关系

以前很多人写：

```cpp
assert(("invalid state", x > 0));
```

利用逗号运算符：

```cpp
("invalid state", x > 0)
```

最终结果仍是：

```cpp
x > 0
```

只是为了带一个字符串。

或者：

```cpp
assert(x > 0 && "invalid state");
```

------

C++26 并没有引入：

```cpp
assert(expr, msg)
```

这样的新语法。

仍然是：

```cpp
assert(expression);
```

只是 expression 现在可以自然包含逗号。

------

### 实现角度

旧实现：

```cpp
#define assert(expr)
```

新实现大致类似：

```cpp
#define assert(...) ...
```

然后内部：

```cpp
(__VA_ARGS__)
```

作为条件。

因此：

```cpp
assert(a, b, c)
```

会被视为：

```cpp
(a, b, c)
```

即合法的逗号表达式。

最终判断结果是：

```cpp
c
```

------

### GCC 16 / Clang 20 / MSVC

实现这个特性后：

```cpp
assert(std::is_same_v<int,long>);
```

即可直接编译。

你可以测试：

```cpp
#include <cassert>
#include <type_traits>

int main()
{
    assert(std::is_same_v<int,int>);
}
```

如果不再要求：

```cpp
assert((std::is_same_v<int,int>));
```

说明标准库已经采用了 C++26 版本的 `assert`。

### 一句话总结

C++26 的 “Making assert() macro user friendly” 本质上是把：

```cpp
assert(expr)
```

升级为基于变参宏的实现，使得：

```cpp
assert(std::is_same_v<int,int>);
assert(std::pair<int,int>{}.first == 0);
assert(std::tuple<int,double>{}.index() == 0);
```

这类包含逗号的表达式终于可以直接书写，不再需要恼人的双括号：

```cpp
assert((expression));
```

这是一个纯易用性改进，没有改变 `assert` 的运行时语义。