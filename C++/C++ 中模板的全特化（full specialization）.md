# C++ 中模板的全特化（full specialization）Explicit/Full template specialization



```C++
#include <string_view>
#include <print>

// Function templates example
template <typename X>
void fun(const X&) { std::println("base template: const X&"); }

// Full specialization must always fully match the base template
template <> void fun<int>(const int&) { std::println("specialization: int"); }
template <> void fun<bool>(const bool&) { std::println("specialization: bool"); }

// Wouldn't compile, argument mismatch
// template<> void fun<double>(double) {}
// Wouldn't compile, return type mismatch
// template<> bool fun<double>(const double&) {}

// Functions can coexist with matching specializations without conflict
// Resolution will prefer functions over templates
// (but prioritize number of implicit coversions over that)
void fun(const int&) { std::println("function: const int&"); }
void fun(const double&) { std::println("function: const double&"); }

// Used as a customization point

// Base template without implementation
template <typename T> struct DoSomething;
// Full specialization for concrete type
template<> struct DoSomething<int> {
    void do_stuff() { std::println("DoSomething<int>"); }
};

// Customization point with a default

// Base template provides the default value
template <typename T>
constexpr std::string_view label = "unknown type";

// Full specializations for concrete types
template <>
constexpr std::string_view label<unsigned> = "unsigned integer";
template <>
constexpr std::string_view label<std::string_view> = "string view";

int main() {
    fun(4.2); // fun(const double&)
    fun(42); // fun(const int&)
    fun(true); // fun<bool>(const bool&) (specialization)
    fun(long{42}); // fun<long>(const long&) (base template)
    std::println("--");

    DoSomething<int>{}.do_stuff(); // OK
    // DoSomething<double>{}.do_stuff(); // Wouldn't compile
    std::println("--");
    
    std::println(label<unsigned>); // "unsigned integer"
    std::println(label<void>); // "unknown type"
}
```



这段代码展示了 C++ 中模板的全特化（full specialization）的用法，包括函数模板、类模板和变量模板的各种场景。它还涉及函数模板与普通函数的重载解析规则。以下是逐步解释。



------

代码概览

- 定义一个函数模板 fun 并为其提供特化和普通函数版本，展示调用解析。
- 定义一个类模板 DoSomething，只为特定类型提供实现。
- 定义一个变量模板 label，通过特化设置特定类型的常量值。

------

关键组件

1. **头文件**

cpp

```cpp
#include <string_view>
```

- <string_view>：提供 std::string_view，用于常量字符串。
- **函数模板 fun**

cpp

```cpp
template <typename X>
void fun(const X&) {}

template <> void fun<int>(const int&) {}
template <> void fun<bool>(const bool&) {}
```

- **基模板**：
  - fun(const X&)：接受任意类型的常量引用。
- **全特化**：
  - fun<int>：特化为 int 类型，参数必须匹配基模板（const int&）。
  - fun<bool>：特化为 bool 类型。
- **限制**：
  - 全特化必须与基模板的参数和返回类型一致：
    - template<> void fun<double>(double)：参数不匹配（非 const double&），无法编译。
    - template<> bool fun<double>(const double&)：返回类型不匹配（void vs bool），无法编译。

**普通函数共存**

cpp

```cpp
void fun(const int&) {}
void fun(const double&) {}
```

- 定义普通函数，与模板共存。
- **重载解析规则**：
  - 普通函数优先于模板函数。
  - 如果参数匹配度相同，优先考虑隐式转换次数。

**调用解析**

cpp

```cpp
fun(4.2);    // fun(const double&)
fun(42);     // fun(const int&)
fun(true);   // fun<bool>(const bool&)
fun(long{42}); // fun<long>(const long&)
```

- **fun(4.2)**：
  - 4.2 是 double，匹配普通函数 fun(const double&)。
- **fun(42)**：
  - 42 是 int，匹配普通函数 fun(const int&)（优先于特化）。
- **fun(true)**：
  - true 是 bool，无普通函数，匹配特化 fun<bool>。
- **fun(long{42})**：
  - long{42} 是 long，无普通函数或特化，使用基模板 fun<long>。
- **类模板 DoSomething**

cpp

```cpp
template <typename T> struct DoSomething;
template<> struct DoSomething<int> {
    void do_stuff() {}
};

DoSomething<int>{}.do_stuff(); // OK
// DoSomething<double>{}.do_stuff(); // Wouldn't compile
```

- **基模板**：
  - DoSomething<T> 未定义实现（声明但无定义）。
- **全特化**：
  - DoSomething<int>：为 int 提供具体实现，包含 do_stuff()。
- **调用**：
  - DoSomething<int>{}.do_stuff()：有效，因为有特化。
  - DoSomething<double>：无效，基模板未定义，链接错误。
- **变量模板 label**

cpp

```cpp
template <typename T>
constexpr std::string_view label = "unknown type";

template <>
constexpr std::string_view label<unsigned> = "unsigned integer";
```

- **基模板**：
  - label<T>：默认值为 "unknown type"。
- **全特化**：
  - label<unsigned>：特化为 "unsigned integer"。
- **结果**：
  - label<unsigned> → "unsigned integer"。
  - label<void> → "unknown type"（无特化，使用默认）。

------

为什么这样工作？

1. **函数模板特化**：
   - 全特化必须匹配基模板的签名。
   - 普通函数优先级高于模板，解析基于最佳匹配。
2. **类模板特化**：
   - 基模板可无定义，仅特化类型可用。
3. **变量模板特化**：
   - 提供类型特定的常量值，未特化的类型使用默认值。
4. **重载解析**：
   - 优先级：普通函数 > 特化 > 基模板。

------

输出

- 无显式输出，但调用和特化行为如下：
  - fun(4.2) 调用普通 fun(const double&)。
  - fun(42) 调用普通 fun(const int&)。
  - fun(true) 调用特化 fun<bool>。
  - fun(long{42}) 调用基模板 fun<long>。
  - label<unsigned> == "unsigned integer"。
  - label<void> == "unknown type"。

------

使用场景

- **函数重载**：
  - 为特定类型定制行为，同时保留通用模板。
- **类模板**：
  - 仅为支持的类型提供实现。
- **常量配置**：
  - 使用变量模板为类型绑定元数据。

------

总结

- fun 展示了函数模板特化与普通函数的共存和解析。
- DoSomething 展示了类模板特化的限制性实现。
- label 展示了变量模板的默认和特化值。
- 代码突出 C++ 模板特化的规则和灵活性。