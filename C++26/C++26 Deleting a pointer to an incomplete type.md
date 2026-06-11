#### Deleting a pointer to an incomplete type

这是一个针对 C++ 语言重大安全改进的重要提案。**是的，C++26 通过了 [P3144R2 提案](https://github.com/cplusplus/papers/issues/1796)****，正式将“删除指向不完整类型的指针（Deleting a pointer to an incomplete type）”从原本的未定义行为（UB）改为了非法的非良构（ill-formed）行为，无法通过编译**。 [1, 2, 3, 4] 

------

## 🔍 历史痛点与安全隐患

在 **C++23 及更早的标准**中，对一个不完整类型（即只有前置声明 `class Foo;`，没有具体定义的类型）的指针执行 `delete`，其标准行为非常具有欺骗性： [5] 

- **标准定义**：只要该类型在补全定义后，**拥有非平凡析构函数（non-trivial destructor）\**或\**类自定义的释放函数（operator delete）**，此处的 `delete` 就会触发**未定义行为（Undefined Behavior, UB）**。
- **现实后果**：编译器在编译到该 `delete` 语句时，因为不知道类的具体内部结构，往往只会静默地生成释放内存的代码（类似 `free`），而**直接跳过析构函数的调用**。这会导致内存泄漏、资源（如文件句柄、锁）无法释放等严重 Bug，而编译器通常只给出一个低级别的警告（如 MSVC 的 [C4150 警告](https://learn.microsoft.com/en-us/cpp/error-messages/compiler-warnings/compiler-warning-level-2-c4150?view=msvc-170)）。 [1, 6, 7, 8, 9] 

------

## 🛠️ C++26 的彻底改变 (P3144R2)

C++ 委员会认为这是一个极度危险且不必要的安全漏洞，为了践行 C++26 提升类型安全（Safety）的核心目标，决定跳过常规的“先弃用（Deprecate）”阶段，**直接在 C++26 中将其升级为编译期错误（ill-formed）**。 [2, 10, 11] 

## ❌ 导致编译失败的典型错误代码 (C++26) [3, 12] 

```cpp
// 仅有前置声明，此时为不完整类型 (Incomplete Type)
class Widget; 

void destroy(Widget* ptr) {
    // C++23: 允许编译（可能报弱警告），但若 Widget 有析构函数则触发 UB
    // C++26: 编译失败！error: deleting pointer to incomplete type 'Widget'
    delete ptr; 
}
```

## 正确代码示例

要修复该编译错误，在调用 `delete` 的翻译单元（.cpp 文件）中，**必须 `#include` 包含该类完整定义的头文件**： [13] 

```cpp
// widget.h
class Widget {
public:
    ~Widget() { /* 释放资源 */ }
};

// widget.cpp
#include "widget.h" // 引入完整定义，使类型 Complete

void destroy(Widget* ptr) {
    delete ptr; // C++26 完美通过，安全调用析构函数并释放内存
}
```

------

## 💡 对现代 C++ 开发（如 pImpl 模式）的影响

如果你经常使用 **pImpl 模式（指针实现模式）** 配合智能指针 `std::unique_ptr`，这个特性的变更会进一步保障你的代码安全。

当你没有在合适的地方提供析构函数时，`std::unique_ptr` 内部的 `std::default_delete` 会触发 `static_assert` 阻止编译。C++26 语言层面的这一改动，让即使是手写 `delete` 的传统代码（或老旧库）也无法隐藏这种潜藏的 UB。

如果对这个提案的演进或者类似 **C++26 移除的其它特性**（例如：P3144 伴随的其它安全清理）感兴趣，我们可以深入聊聊！

[1] [https://www.open-std.org](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2024/p3144r2.pdf)

[2] [https://www.sandordargo.com](https://www.sandordargo.com/blog/2025/03/12/cpp26-removing-language-features)

earn.microsoft.com](https://learn.microsoft.com/en-us/cpp/error-messages/compiler-warnings/compiler-warning-level-2-c4150?view=msvc-170)