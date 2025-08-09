

#  C++ 中名称查找（name lookup）和作用域解析



```C++
#include <iostream>
constexpr auto printer = []{ std::cout << "The global one.\n"; };

struct Printer {
    void printer() { std::cout << "The internal one.\n"; }
};

struct X : Printer {
  	// Printer::printer found by non-ADL lookup
    void fun() { printer(); }
};

template <typename T> struct Y : T {
  	// Non-dependent name, bound to ::printer
    void fun() { printer(); }
};

template <typename T> struct Z : T {
  	// Pulling Printer::printer into current scope
    using T::printer;
    // Early binding will find Z::printer, which a dependent name
    // Non-ADL lookup will then also find Z::printer,
    // which is resolved into Printer::printer
    void fun() { printer(); }
};

template <typename T> struct W : T {
    // Alternative approach, dependent name because W derives from T.
    void fun() { this->printer(); }
};

int main() {
    X{}.fun();
    Y<Printer>{}.fun();
    Z<Printer>{}.fun();
    W<Printer>{}.fun();
}
```



这段代码展示了 C++ 中名称查找（name lookup）和作用域解析的复杂性，特别是非依赖名称、依赖名称以及 ADL（Argument-Dependent Lookup，参数依赖查找）的影响。

代码定义了一个全局 printer lambda 和一个 Printer 结构体，包含成员函数 printer，然后通过不同的类（X、Y、Z、W）展示了在不同情况下调用 printer 的行为。

以下是逐步解释。

------

代码概览

- 定义全局 constexpr lambda printer 和结构体 Printer。
- 通过继承和模板，测试在不同作用域中 printer() 的解析。
- 在 main 中调用每个类的 fun()，观察输出。

------

关键组件

1. **全局 printer**

cpp

```cpp
constexpr auto printer = []{ std::cout << "The global one.\n"; };
```

- 定义一个全局 lambda，输出 "The global one."。
- 位于全局命名空间（::printer）。
- **Printer 结构体**

cpp

```cpp
struct Printer {
    void printer() { std::cout << "The internal one.\n"; }
};
```

- 定义成员函数 printer，输出 "The internal one."。
- **X 类**

cpp

```cpp
struct X : Printer {
    void fun() { printer(); }
};
```

- **继承**：X 继承自 Printer，拥有 Printer::printer。
- **fun()**：
  - 调用 printer()，无限定符。
  - **非 ADL 查找**（普通名称查找）：
    - 从当前作用域（X）开始，找到基类 Printer 的 printer。
    - 不涉及 ADL，因为 printer() 无参数。
- **结果**：
  - 调用 Printer::printer，输出 "The internal one."。
- **Y 模板类**

cpp

```cpp
template <typename T> struct Y : T {
    void fun() { printer(); }
};
```

- **继承**：Y<Printer> 继承 Printer。
- **fun()**：
  - printer 是非依赖名称（non-dependent name），不依赖模板参数 T。
  - 在模板定义时解析（early binding）。
  - **查找**：
    - 从 Y 作用域开始，未找到 printer。
    - 检查外围作用域，找到全局 ::printer。
    - 基类 T（Printer）的 printer 未考虑，因为名称非依赖。
- **结果**：
  - 调用全局 ::printer，输出 "The global one."。
- **Z 模板类**

cpp

```cpp
template <typename T> struct Z : T {
    using T::printer;
    void fun() { printer(); }
};
```

- **继承**：Z<Printer> 继承 Printer。
- **using T::printer**：
  - 将基类 T 的 printer 引入 Z 作用域，成为依赖名称（dependent name）。
- **fun()**：
  - printer 是依赖名称，解析延迟到实例化时。
  - **查找**：
    - 在 Z<Printer> 实例化时，using 使 Printer::printer 成为 Z::printer。
    - 非 ADL 查找在 Z 作用域找到 Z::printer（即 Printer::printer）。
- **结果**：
  - 调用 Printer::printer，输出 "The internal one."。
- **W 模板类**

cpp

```cpp
template <typename T> struct W : T {
    void fun() { this->printer(); }
};
```

- **继承**：W<Printer> 继承 Printer。
- **fun()**：
  - this->printer() 显式限定为成员调用。
  - printer 是依赖名称（依赖 T 的成员），解析延迟到实例化。
  - **查找**：
    - this 指向 W，查找成员 printer。
    - 在基类 Printer 中找到 printer。
- **结果**：
  - 调用 Printer::printer，输出 "The internal one."。
- **main 函数**

cpp

```cpp
int main() {
    X{}.fun();          // "The internal one."
    Y<Printer>{}.fun(); // "The global one."
    Z<Printer>{}.fun(); // "The internal one."
    W<Printer>{}.fun(); // "The internal one."
}
```

- 创建临时对象并调用 fun()，观察输出。

------

输出

```text
The internal one.
The global one.
The internal one.
The internal one.
```

------

为什么这样工作？

1. **名称查找规则**：
   - **非依赖名称**（如 Y::fun 中的 printer）：
     - 在定义时解析，优先外围作用域（::printer）。
   - **依赖名称**（如 Z 和 W 中的 printer）：
     - 延迟到实例化时解析，考虑基类作用域。
2. **using 声明**：
   - Z 中将 T::printer 引入当前作用域，使其可直接调用。
3. **this->**：
   - W 中强制成员查找，避免全局作用域干扰。
4. **ADL**：
   - 此例中无影响，因 printer() 无参数。

------

使用场景

- **模板编程**：
  - 处理基类成员访问，避免名称冲突。
- **接口设计**：
  - 控制全局和成员函数的优先级。
- **调试**：
  - 理解名称解析规则。

------

总结

- X：普通继承，非 ADL 找到 Printer::printer。
- Y：非依赖名称绑定到全局 ::printer。
- Z：通过 using 引入依赖名称，解析为 Printer::printer。
- W：使用 this-> 显式调用 Printer::printer。
- 代码展示了 C++ 名称查找的复杂性和灵活性。