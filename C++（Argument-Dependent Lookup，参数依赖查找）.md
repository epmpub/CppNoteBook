# （Argument-Dependent Lookup，参数依赖查找）

这段代码涉及 C++ 中的 **ADL（Argument-Dependent Lookup，参数依赖查找）** 以及命名空间、函数对象（constexpr inline auto fun）和函数重载的交互。让我逐步解释这段代码的工作原理。

------

代码概览

- 代码展示了两个命名空间 SomeNamespace 和 OtherNamespace，分别定义了结构体和函数/函数对象。
- SomeNamespace::fun 是一个 constexpr inline 的 lambda（函数对象），而 OtherNamespace::fun 是一个普通函数模板。
- main 函数中测试了如何调用这些函数，并展示了 ADL 的行为。

------

```c++
#include <iostream>

namespace SomeNamespace {
    struct X {};
    // A typical use case, non-function symbols do not participate in ADL.
    constexpr inline auto fun = [](auto&&) { std::cout << "SomeNamespace::fun()\n"; };
    // constexpr not required, but advisable
}

namespace OtherNamespace {
    struct Y {};
    void fun(auto&&) { std::cout << "OtherNamespace::fun()\n"; };
}

int main() {
    SomeNamespace::X x;
    // fun(x); // Would not compile, does not participate in ADL.
    SomeNamespace::fun(x); // OK

    using SomeNamespace::fun;
    OtherNamespace::Y y;
    fun(y); // calls SomeNamespace::fun
    // OtherNamespace::fun is only visible to ADL
}
```



关键组件

1. **SomeNamespace 命名空间**

cpp

```cpp
namespace SomeNamespace {
    struct X {};
    constexpr inline auto fun = [](auto&&) {};
}
```

- **struct X**：一个简单的空结构体，用于测试。
- **constexpr inline auto fun**：
  - 这是一个 lambda 表达式，定义为命名空间作用域的变量。
  - [](auto&&) 表示它接受任意类型的参数（通过通用引用）。
  - constexpr 表示这个函数对象可以在编译时使用（虽然这里不是必须的，但建议使用以提高灵活性）。
  - inline 表示这个变量的定义是内联的，避免多重定义问题。
  - fun 是一个函数对象（本质上是一个具有 operator() 的对象），而不是普通的函数。
- **关键点**：因为 fun 是一个变量（而非函数），它不会参与 ADL（参数依赖查找）。ADL 只适用于函数名，而不适用于函数对象。
- **OtherNamespace 命名空间**

cpp

```cpp
namespace OtherNamespace {
    struct Y {};
    void fun(auto&&) {}
}
```

- **struct Y**：另一个简单的空结构体。
- **void fun(auto&&)**：
  - 这是一个普通的函数模板，接受任意类型的参数。
  - 因为它是函数（而非变量），它可以参与 ADL。
- **关键点**：当调用 fun 时，如果参数的类型与 OtherNamespace 相关（例如 Y），ADL 会查找这个函数。
- **main 函数**

cpp

```cpp
int main() {
    SomeNamespace::X x;
    // fun(x); // Would not compile, does not participate in ADL.
    SomeNamespace::fun(x); // OK

    using SomeNamespace::fun;
    OtherNamespace::Y y;
    fun(y); // calls SomeNamespace::fun
    // OtherNamespace::fun is only visible to ADL
}
```

- **第一部分：SomeNamespace::X x;**
  - 创建一个 SomeNamespace::X 类型的对象 x。
  - **fun(x);**（注释掉的代码）：
    - 这行代码无法编译。
    - 原因：fun 未在全局作用域定义，且 SomeNamespace::fun 是一个变量（函数对象），不会通过 ADL 被找到。
    - ADL 只适用于函数名，并且会查找与参数类型（如 X）关联的命名空间中的函数。但 SomeNamespace::fun 是一个变量，不是函数，因此不参与 ADL。
  - **SomeNamespace::fun(x);**：
    - 这行代码可以正常工作。
    - 通过显式限定命名空间 SomeNamespace::，直接访问 fun 变量，并调用其 operator()（lambda 的调用方式）。
    - 这里不需要 ADL，因为它是显式调用。
- **第二部分：using SomeNamespace::fun;**
  - 这将 SomeNamespace::fun 引入当前作用域（main 函数的作用域）。
  - 现在，fun 在 main 中是可见的，并且可以直接调用。
- **第三部分：OtherNamespace::Y y; fun(y);**
  - 创建一个 OtherNamespace::Y 类型的对象 y。
  - **fun(y);**：
    - 这调用的是 SomeNamespace::fun，而不是 OtherNamespace::fun。
    - **原因**：
      1. using SomeNamespace::fun; 已经将 SomeNamespace::fun 引入当前作用域，因此它是直接可见的。
      2. OtherNamespace::fun 是一个函数模板，可以通过 ADL 找到（因为 y 是 OtherNamespace::Y 类型，ADL 会查找 OtherNamespace 中的函数）。
      3. 但是，C++ 的名称查找规则优先考虑当前作用域中显式引入的名称（SomeNamespace::fun），而不是通过 ADL 找到的候选者（OtherNamespace::fun）。
    - 因此，fun(y) 调用的是 SomeNamespace::fun 的 lambda，而不是 OtherNamespace::fun。
- **注释说明**：
  - "OtherNamespace::fun is only visible to ADL" 表示，如果没有 using SomeNamespace::fun，调用 fun(y) 会通过 ADL 找到 OtherNamespace::fun，因为 y 的类型与 OtherNamespace 相关。

------

ADL（参数依赖查找）的工作原理

- ADL 是 C++ 中用于查找函数的一种机制。
- 当调用一个未限定名称的函数（如 fun(x)）时，编译器会：
  1. 检查当前作用域中是否有匹配的函数。
  2. 通过 ADL 检查与参数类型相关的命名空间中的函数。
- 对于 SomeNamespace::X x，如果 fun 是普通函数且定义在 SomeNamespace 中，fun(x) 可以通过 ADL 找到。但在这里，fun 是一个变量，不参与 ADL。
- 对于 OtherNamespace::Y y，OtherNamespace::fun 可以通过 ADL 找到，但被 using 引入的 SomeNamespace::fun 覆盖。

------

输出和行为

- **SomeNamespace::fun(x);**：成功调用 lambda，无输出（因为 lambda 体为空）。
- **fun(y);**：调用 SomeNamespace::fun，而不是 OtherNamespace::fun，无输出。
- 如果去掉 using SomeNamespace::fun;，则 fun(y) 会通过 ADL 调用 OtherNamespace::fun。

------

为什么这样设计？

1. **SomeNamespace::fun 作为变量**：
   - 使用 lambda 定义函数对象是一种现代 C++ 技术，提供灵活性（例如可以在编译时使用 constexpr）。
   - 但作为变量，它不会触发 ADL，这是一种有意限制，可能用于避免意外的名称冲突。
2. **OtherNamespace::fun 作为函数**：
   - 普通函数模板参与 ADL，这是 C++ 的传统行为，便于扩展和重载。
3. **using 的优先级**：
   - C++ 规则确保显式引入的名称优先于 ADL，避免 ADL 带来的不可预测性。

------

总结

- SomeNamespace::fun 是一个函数对象，不参与 ADL，必须显式调用或通过 using 引入。
- OtherNamespace::fun 是一个函数模板，可以通过 ADL 找到，但被当前作用域中的 SomeNamespace::fun 覆盖。
- 这段代码展示了 ADL 的局限性以及 using 对名称查找的影响。
- 如果你想测试 OtherNamespace::fun，可以去掉 using SomeNamespace::fun;，然后 fun(y) 会调用它。