# C++23 std::forward_like

在 C++23 之前正确转发复合类型成员的值类别非常麻烦。

幸运的是，C++23 引入了*std::forward_like*，这使得这个操作变得简单得多。

使用“推断此”功能时，std::forward_like 也是必不可少的*。*



```C++
#include <utility>
#include <print>

struct Wrapper {
    int member;
};

void fun(const int&) {
    std::println("const int&");
}
void fun(int&) {
    std::println("int&");
}
void fun(int&&) {
    std::println("int&&");
}

// Only calls fun(int&) or fun(const int&)
void extract1(auto&& wrapper) {
    fun(wrapper.member);
}

// Correct, but cumbersome
void extract2(auto&& wrapper) {
    if constexpr (std::is_rvalue_reference_v<decltype(wrapper)>) {
        fun(std::move(wrapper.member));
    } else {
        fun(wrapper.member);
    }
}

// Using C++23 forward_like
void extract3(auto&& wrapper) {
    fun(std::forward_like<decltype(wrapper)>(wrapper.member));
}

// Typical use case
struct MyType {
    auto&& get(this auto&& self) {
        // One getter variant covering all value categories
        return std::forward_like<decltype(self)>(self.data);
    }
    int data;
};

int main() {
    Wrapper w;
    extract1(w); // calls fun(int&)
    extract1(std::as_const(w)); // calls fun(const int&)
    extract1(Wrapper{}); // calls fun(int&)
    std::println("--");
    
    extract2(w); // calls fun(int&)
    extract2(std::as_const(w)); // calls fun(const int&)
    extract2(Wrapper{}); // calls fun(int&&)
    std::println("--");

    extract3(w); // calls fun(int&)
    extract3(std::as_const(w)); // calls fun(const int&)
    extract3(Wrapper{}); // calls fun(int&&)
}
```

这段代码展示了 C++ 中值类别（value category）转发机制的演进，特别是 C++23 引入的 std::forward_like（来自 <utility>），并通过三个函数（extract1、extract2、extract3）对比了不同转发策略的效果。以下是逐步解释。

------

代码概览

- 定义 fun 的三个重载，分别处理 const int&、int& 和 int&&。
- 实现三种提取函数，展示如何转发 Wrapper 的成员 member。
- 使用 MyType 展示 std::forward_like 在 this 参数中的典型用法。

------

关键组件

1. **头文件**

cpp

```cpp
#include <utility>
#include <print>
```

- <utility>：提供 std::forward_like 和 std::move。
- <print>：C++23 的 std::println 用于输出。
- **函数重载**

cpp

```cpp
void fun(const int&) { std::println("const int&"); }
void fun(int&) { std::println("int&"); }
void fun(int&&) { std::println("int&&"); }
```

- **作用**：
  - 分别处理常量左值引用、左值引用和右值引用。
  - 输出反映参数的值类别。
- **extract1：简单调用**

cpp

```cpp
void extract1(auto&& wrapper) {
    fun(wrapper.member);
}
```

- **auto&& wrapper**：
  - 通用引用，接受任意值类别（左值或右值）。
- **行为**：
  - wrapper.member 是成员访问，总是左值（无论 wrapper 是左值还是右值）。
  - 只调用 fun(int&) 或 fun(const int&)。
- **限制**：
  - 无法转发右值类别，丢失 int&& 的可能性。
- **extract2：手动转发**

cpp

```cpp
void extract2(auto&& wrapper) {
    if constexpr (std::is_rvalue_reference_v<decltype(wrapper)>) {
        fun(std::move(wrapper.member));
    } else {
        fun(wrapper.member);
    }
}
```

- **逻辑**：
  - 使用 std::is_rvalue_reference_v 检查 wrapper 是否为右值引用。
  - 若为右值，std::move(wrapper.member) 转为 int&&。
  - 否则，保持左值调用。
- **行为**：
  - 正确转发所有值类别。
- **缺点**：
  - 代码繁琐，需显式条件分支。
- **extract3：使用 std::forward_like**

cpp

```cpp
void extract3(auto&& wrapper) {
    fun(std::forward_like<decltype(wrapper)>(wrapper.member));
}
```

- **std::forward_like**：
  - C++23 新增，原型：forward_like<T>(x)。
  - 根据 T 的值类别（decltype(wrapper)）转发 x：
    - T 是左值引用（X& 或 const X&）：x 保持左值。
    - T 是右值引用（X&&）：x 转为右值。
- **行为**：
  - 简洁地实现完美转发，匹配 wrapper 的值类别。
- **MyType：典型用例**

cpp

```cpp
struct MyType {
    auto&& get(this auto&& self) {
        return std::forward_like<decltype(self)>(self.data);
    }
    int data;
};
```

- **this auto&& self**：
  - C++23 显式 this 参数，匹配调用者的值类别。
- **get**：
  - 返回 data，根据 self 的类别转发（左值或右值）。
- **作用**：
  - 单一函数处理所有值类别。
- **main 函数**

cpp

```cpp
Wrapper w;
extract1(w);              // fun(int&)
extract1(std::as_const(w)); // fun(const int&)
extract1(Wrapper{});       // fun(int&)
extract2(w);              // fun(int&)
extract2(std::as_const(w)); // fun(const int&)
extract2(Wrapper{});       // fun(int&&)
extract3(w);              // fun(int&)
extract3(std::as_const(w)); // fun(const int&)
extract3(Wrapper{});       // fun(int&&)
```

- **w**：左值，member 调用 fun(int&)。
- **std::as_const(w)**：常量左值，member 调用 fun(const int&)。
- **Wrapper{}**：右值：
  - extract1：member 是左值，调用 fun(int&)。
  - extract2 和 extract3：正确转发为 int&&。

------

输出

```text
int&
const int&
int&
--
int&
const int&
int&&
--
int&
const int&
int&&
```

------

为什么这样工作？

1. **extract1**：
   - wrapper.member 总是左值，限制了转发能力。
2. **extract2**：
   - 手动检查值类别，繁琐但正确。
3. **std::forward_like**：
   - 根据 wrapper 的类型动态调整 member 的类别：
     - Wrapper& → int&。
     - const Wrapper& → const int&。
     - Wrapper&& → int&&。
4. **MyType::get**：
   - 展示 std::forward_like 在方法中的实用性。

------

使用场景

- **完美转发成员**：
  - 在模板中保留值类别。
- **简化代码**：
  - 替代繁琐的 if constexpr。
- **通用方法**：
  - 处理所有调用场景。

------

总结

- extract1 只支持左值转发。
- extract2 手动实现完整转发。
- extract3 使用 std::forward_like 简洁高效。
- 代码展示了 C++23 新特性对转发机制的改进。