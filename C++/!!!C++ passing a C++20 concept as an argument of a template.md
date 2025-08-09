

# C++ concept as an argument of a template

```C++
#include <concepts>

// Indirect concept check through a lambda
template <auto wrapper, typename T>
concept satisfies_wrapped_concept = requires {
    { wrapper.template operator()<T>() };
};

// Template parametrized using a wrapped concept and a type
template <auto WrappedConcept, typename T>
// Limit T to types that satisfy the passed in concept
requires satisfies_wrapped_concept<WrappedConcept, T>
struct Box {};

// Demonstration of use:
template <typename T>
// Box parametrized for integral types
// A templated lambda, with the template argument constrained with a concept
using BoxForIntegrals = Box<[]<std::integral>{},T>;
template <typename T>
// Box parametrized for floating point types
using BoxForFloats = Box<[]<std::floating_point>{},T>;

int main() {
    BoxForIntegrals<int> a; // OK
    // BoxForIntegrals<float> b; // Wouldn't compile
    BoxForFloats<float> c; // OK
    // BoxForFloats<int> d; // Wouldn't compile
}
```

这段代码展示了 C++20 中 <concepts> 头文件的应用，通过定义一个间接概念（concept）来约束模板参数。代码使用模板化的 lambda 和 requires 子句实现类型约束，并通过 Box 模板展示其用法。以下是逐步解释。

------

代码概览

- 定义 satisfies_wrapped_concept，检查类型是否满足通过 lambda 包装的概念。
- 定义 Box 模板，结合概念和类型约束。
- 使用 BoxForIntegrals 和 BoxForFloats 分别限制整数和浮点类型。

------

关键组件

1. **头文件**

cpp

```cpp
#include <concepts>
```

- <concepts>：提供标准概念（如 std::integral、std::floating_point）和自定义概念支持。
- **satisfies_wrapped_concept 定义**

cpp

```cpp
template <auto wrapper, typename T>
concept satisfies_wrapped_concept = requires {
    { wrapper.template operator()<T>() };
};
```

- **模板参数**：
  - auto wrapper：非类型模板参数，表示一个可调用对象（如 lambda）。
  - typename T：待检查的类型。
- **requires 子句**：
  - 检查 wrapper.operator()<T>() 是否有效。
  - wrapper 是一个模板化的 lambda，调用时需用 .template 显式指定。
- **作用**：
  - 间接验证 T 是否满足 wrapper 内部定义的概念。
- **Box 模板定义**

cpp

```cpp
template <auto WrappedConcept, typename T>
requires satisfies_wrapped_concept<WrappedConcept, T>
struct Box {};
```

- **模板参数**：
  - auto WrappedConcept：包装的概念（如 lambda）。
  - typename T：待约束的类型。
- **requires 子句**：
  - 限制 T 必须满足 WrappedConcept 定义的条件。
- **作用**：
  - 一个简单的容器，类型受概念约束。
- **使用示例**

cpp

```cpp
template <typename T>
using BoxForIntegrals = Box<[]<std::integral>{}, T>;
template <typename T>
using BoxForFloats = Box<[]<std::floating_point>{}, T>;
```

- **BoxForIntegrals**：
  - 使用 lambda []<std::integral>{} 约束 T 为整数类型。
  - 空 lambda 表示仅检查概念，无具体操作。
- **BoxForFloats**：
  - 使用 lambda []<std::floating_point>{} 约束 T 为浮点类型。
- **别名模板**：
  - 简化 Box 的实例化，固定概念部分。
- **main 函数**

cpp

```cpp
int main() {
    BoxForIntegrals<int> a;   // OK
    // BoxForIntegrals<float> b; // Wouldn't compile
    BoxForFloats<float> c;    // OK
    // BoxForFloats<int> d;    // Wouldn't compile
}
```

- **BoxForIntegrals<int>**：
  - int 满足 std::integral，编译通过。
- **BoxForIntegrals<float>**：
  - float 不满足 std::integral，编译失败。
- **BoxForFloats<float>**：
  - float 满足 std::floating_point，编译通过。
- **BoxForFloats<int>**：
  - int 不满足 std::floating_point，编译失败。

------

为什么这样工作？

1. **概念间接检查**：
   - satisfies_wrapped_concept 通过 lambda 的模板参数约束验证类型。
   - requires 子句确保 lambda 对 T 的调用有效。
2. **模板化的 Lambda**：
   - []<std::integral>{} 定义一个约束为整数类型的无操作 lambda。
   - 类似地，[]<std::floating_point>{} 约束为浮点类型。
3. **Box 的约束**：
   - requires 子句将概念检查嵌入模板，确保类型匹配。
4. **编译期验证**：
   - 不满足约束的实例化会导致编译错误。

------

输出

- 无运行时输出，代码仅验证编译期行为。

------

使用场景

- **类型安全**：
  - 限制模板参数到特定类型集合。
- **泛型编程**：
  - 结合概念和 lambda 提供灵活约束。
- **代码可读性**：
  - 通过别名清晰表达意图（如整数专用、浮点专用）。

------

总结

- satisfies_wrapped_concept 检查类型是否满足 lambda 包装的概念。
- Box 使用此概念约束模板参数。
- BoxForIntegrals 和 BoxForFloats 分别限制为整数和浮点类型。
- 代码展示了 C++20 概念的强大功能和灵活性。