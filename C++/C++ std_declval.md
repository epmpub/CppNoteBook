# std::declval 

这段代码展示了 C++ 中 <utility> 提供的 std::declval 的用法，用于提供假想值（T&&），仅用于类型推导，

不执行构造。

用于在类型推导中处理不可默认构造的类型。

代码通过 decltype 和 std::is_same_v 验证函数和方法的返回类型，突出 std::declval 在避免构造对象时的作用。

以下是逐步解释。

------

```C++
#include <utility>
#include <type_traits>
#include <functional>

template <typename T>
auto fun(T t) { return t+1.0; }

struct MyInt {
    MyInt() = delete; // Not default constructible
    friend int operator+(const MyInt&, double) { return {}; }
    int method() { return {}; }
};

int main() {
    // Get the result type of fun for double and int, all OK
    static_assert(
        std::is_same_v<decltype(fun(double{})), double>);
    static_assert(
        std::is_same_v<decltype(fun(int{})), double>);
    // But we cannot spell the same for MyInt
    // static_assert(
    //    std::is_same_v<decltype(fun(MyInt{})), int>); // Wouldn't compile

    // std::declval allows us to declare a value without creating it
    static_assert(
        std::is_same_v<decltype(fun(std::declval<MyInt>())), int>);

    // Same for methods
    // static_assert(
    //    std::is_same_v<decltype(MyInt{}.method()), int>); // Wouldn't compile
    static_assert(
        std::is_same_v<decltype(std::declval<MyInt>().method()), int>); // OK
}
```

代码概览

- 定义了一个模板函数 fun 和一个不可默认构造的类型 MyInt。
- 使用 decltype 检查函数和方法的返回类型。
- 展示了 std::declval 如何解决无法构造对象的问题。

------

关键组件

1. **头文件**

cpp

```cpp
#include <utility>
#include <type_traits>
#include <functional>
```

- <utility>：提供 std::declval。
- <type_traits>：提供 std::is_same_v，比较类型。
- <functional>：未直接使用，但常与函数对象相关。
- **模板函数 fun**

cpp

```cpp
template <typename T>
auto fun(T t) { return t + 1.0; }
```

- **定义**：
  - 接受类型 T 的参数 t，返回 t + 1.0。
  - 返回类型由 auto 推导，依赖 + 的结果。
- **行为**：
  - 对于内置类型（如 double、int），返回 double。
  - 对于自定义类型，依赖重载的 operator+。
- **MyInt 类型**

cpp

```cpp
struct MyInt {
    MyInt() = delete; // Not default constructible
    friend int operator+(const MyInt&, double) { return {}; }
    int method() { return {}; }
};
```

- **MyInt()**：
  - 默认构造函数被删除，无法用 {} 构造。
- **operator+**：
  - 友元函数，定义 MyInt + double 返回 int。
- **method()**：
  - 返回 int，模拟成员函数。
- **fun 的类型推导**

cpp

```cpp
static_assert(
    std::is_same_v<decltype(fun(double{})), double>);
static_assert(
    std::is_same_v<decltype(fun(int{})), double>);
```

- **fun(double{})**：
  - double{} 构造值为 0.0，fun 返回 0.0 + 1.0，类型为 double。
- **fun(int{})**：
  - int{} 构造值为 0，0 + 1.0 提升为 double，返回类型为 double。
- **验证**：
  - 两种情况均通过，类型正确。
- **MyInt 的问题**

cpp

```cpp
// static_assert(
//    std::is_same_v<decltype(fun(MyInt{})), int>); // Wouldn't compile
```

- **问题**：
  - MyInt{} 尝试调用默认构造函数，但被删除，无法编译。
- **目的**：
  - 展示无法直接用 decltype 测试 fun(MyInt)。
- **std::declval 解决**

cpp

```cpp
static_assert(
    std::is_same_v<decltype(fun(std::declval<MyInt>())), int>);
```

- **std::declval<T>()**：
  - 返回类型 T&& 的假想值，不实际构造对象。
  - 用于类型推导，避免构造要求。
- **fun(std::declval<MyInt>())**：
  - 假设传入 MyInt&&，调用 fun，触发 operator+(MyInt&, double)。
  - 返回类型为 int。
- **验证**：
  - 通过，确认返回类型为 int。
- **方法类型推导**

cpp

```cpp
// static_assert( 
//    std::is_same_v<decltype(MyInt{}.method()), int>);

static_assert(
    std::is_same_v<decltype(std::declval<MyInt>().method()), int>);
```

- **MyInt{}.method()**：
  - 需要构造 MyInt，但默认构造函数删除，无法编译。
- **std::declval<MyInt>().method()**：
  - std::declval<MyInt>() 返回 MyInt&&。
  - 调用 method()，返回 int。
- **验证**：
  - 通过，确认返回类型为 int。

------

为什么这样工作？

1. **decltype**：
   - 推导表达式的类型，但要求表达式合法。
   - 对于不可构造类型，直接使用会失败。
2. **std::declval**：
   - 提供假想值（T&&），仅用于类型推导，不执行构造。
   - 解决无法实例化的问题。
3. **类型推导**：
   - fun 的返回类型依赖参数类型和操作符。
   - MyInt 的 operator+ 定义了具体行为。

------

输出

- 无运行时输出，仅编译期验证通过。

------

使用场景

- **模板元编程**：
  - 检查函数或方法的返回类型。
- **类型设计**：
  - 处理不可默认构造的类型。
- **静态检查**：
  - 确保接口符合预期。

------

总结

- std::declval 解决了 MyInt 不可构造的问题。
- decltype(fun(std::declval<MyInt>())) 推导出 int。
- decltype(std::declval<MyInt>().method()) 确认方法返回 int。
- 代码展示了 std::declval 在类型推导中的关键作用。