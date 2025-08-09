

# std::invoke_r 指定返回值类型，避免悬垂引用

```C++
#include <functional>

struct Box {
    int value;
};

struct Base {} base;
struct Derived : Base {} derived;
 
Base& fun(int) { return base; }
Derived& fun(double) { return derived; }

int main() {
    Box x{42};
    auto&& a = std::invoke(&Box::value, x); // OK
    auto&& b = std::invoke(&Box::value, Box{42}); // Bad, danling reference

    // std::invoke_r allows the return type to be specified
    auto&& c = std::invoke_r<int>(&Box::value, Box{42}); // OK

    // void is also valid, discarding any potential result
    std::invoke_r<void>(&Box::value, Box{42});

    // Collapsing covariant return types

    // Wrapper for the overload set
    auto wrapped = [](auto arg) -> decltype(auto) { return fun(arg); };

    auto& i1 = std::invoke(wrapped, 42); // calls fun(int)
    // decltype(i1) == Base&
    static_assert(std::is_same_v<decltype(i1), Base&>);

    auto& i2 = std::invoke(wrapped, 4.2); // calls fun(double)
    // decltype(i2) == Derived&
    static_assert(std::is_same_v<decltype(i2), Derived&>);

    auto& i3 = std::invoke_r<Base&>(wrapped, 42); // calls fun(int)
    // decltype(i3) == Base&
    static_assert(std::is_same_v<decltype(i3), Base&>);

    auto& i4 = std::invoke_r<Base&>(wrapped, 4.2); // calls fun(double)
    // decltype(i4) == Base&
    static_assert(std::is_same_v<decltype(i4), Base&>);
}

```

这段代码展示了 C++ 中 <functional> 头文件提供的 std::invoke 和 std::invoke_r 函数，用于调用成员函数或普通函数，并处理返回值类型。

代码通过示例展示了它们的基本用法、悬垂引用问题以及如何处理协变返回类型（covariant return types）。

以下是逐步解释。

------

代码概览

- 使用 std::invoke 调用 Box 的成员变量。
- 使用 std::invoke_r 指定返回值类型，避免悬垂引用。
- 使用 lambda 包装重载函数，展示协变返回类型的处理。

------

关键组件

1. **头文件**

cpp

```cpp
#include <functional>
```

- <functional>：提供 std::invoke 和 std::invoke_r。
- **结构体和函数定义**

cpp

```cpp
struct Box {
    int value;
};
struct Base {} base;
struct Derived : Base {} derived;
Base& fun(int) { return base; }
Derived& fun(double) { return derived; }
```

- **Box**：包含成员 value。
- **Base 和 Derived**：展示继承关系。
- **fun**：重载函数，返回引用：
  - fun(int) 返回 Base&。
  - fun(double) 返回 Derived&（协变返回类型）。
- **std::invoke 示例**

cpp

```cpp
Box x{42};
auto&& a = std::invoke(&Box::value, x); // OK
auto&& b = std::invoke(&Box::value, Box{42}); // Bad, dangling reference
```

- **std::invoke**：
  - 原型：invoke(func, args...)。
  - 调用可调用对象（如成员指针），返回其结果。
- **a**：
  - 调用 x.value，x 是左值，返回 int&。
  - auto&& 推导为 int&，绑定到 x.value。
- **b**：
  - 调用临时对象 Box{42}.value，返回 int&。
  - 临时对象在表达式结束后销毁，b 成为悬垂引用（dangling reference）。
- **问题**：
  - 未指定返回类型可能导致不安全。
- **std::invoke_r 示例**

cpp

```cpp
auto&& c = std::invoke_r<int>(&Box::value, Box{42}); // OK
std::invoke_r<void>(&Box::value, Box{42});
```

- **std::invoke_r**：
  - 原型：invoke_r<R>(func, args...)。
  - 指定返回类型 R，将结果转换为该类型。
- **c**：
  - 调用 Box{42}.value，返回 int（值类型）。
  - auto&& 推导为 int&&，绑定到临时值，避免悬垂引用。
- **void**：
  - 调用并丢弃结果，无返回值。
- **优势**：
  - 显式控制返回类型，安全处理临时对象。
- **协变返回类型处理**

cpp

```cpp
auto wrapped = [](auto arg) -> decltype(auto) { return fun(arg); };
auto& i1 = std::invoke(wrapped, 42);
auto& i2 = std::invoke(wrapped, 4.2);
auto& i3 = std::invoke_r<Base&>(wrapped, 42);
auto& i4 = std::invoke_r<Base&>(wrapped, 4.2);
```

- **wrapped**：
  - Lambda 包装 fun 的重载集。
  - decltype(auto) 保留 fun 的返回类型。
- **i1**：
  - wrapped(42) 调用 fun(int)，返回 Base&。
  - auto& 推导为 Base&。
- **i2**：
  - wrapped(4.2) 调用 fun(double)，返回 Derived&。
  - auto& 推导为 Derived&。
- **i3**：
  - std::invoke_r<Base&> 调用 fun(int)，返回 Base&。
  - 类型强制为 Base&。
- **i4**：
  - std::invoke_r<Base&> 调用 fun(double)，返回 Derived&，但强制转换为 Base&。
  - 协变类型折叠到基类引用。
- **验证**：
  - static_assert 确认类型匹配。

------

为什么这样工作？

1. **std::invoke**：
   - 直接调用成员或函数，返回原始类型。
   - 对临时对象可能产生悬垂引用。
2. **std::invoke_r**：
   - 指定返回类型，转换结果，避免未定义行为。
3. **协变返回**：
   - fun 的重载返回不同类型。
   - std::invoke_r 统一返回类型，处理继承关系。
4. **decltype(auto)**：
   - 保留 lambda 的返回类型，反映重载选择。

------

输出

- 无运行时输出，代码验证编译期行为。

------

使用场景

- **成员访问**：
  - 调用成员变量或函数。
- **临时对象**：
  - 使用 invoke_r 安全处理返回值。
- **重载集**：
  - 包装函数集，统一返回类型。

------

总结

- std::invoke(&Box::value, x) 返回 int&，临时对象导致悬垂引用。
- std::invoke_r<int> 返回值类型，避免问题。
- wrapped 和 invoke_r<Base&> 处理协变返回，统一为 Base&。
- 代码展示了 invoke 和 invoke_r 的功能和安全性。