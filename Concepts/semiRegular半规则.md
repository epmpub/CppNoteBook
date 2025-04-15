# C++ SemiRegular

在 C++ 中，**SemiRegular** 是一个类型概念（concept），主要用于描述一种类型所满足的属性集合。这些属性确保类型可以被“合理地”使用，例如在容器、算法或其他通用代码中。它是 C++ 类型理论中的一个重要分类，尤其在讨论模板编程和标准库设计时常见。

SemiRegular 的定义

一个类型 T 被认为是 **SemiRegular**，如果它满足以下要求：

1. **可默认构造**（Default Constructible）：可以用 T() 创建对象。

   cpp

   ```cpp
   T t{}; // 或 T t = T();
   ```

2. **可复制构造**（Copy Constructible）：可以用一个 T 对象初始化另一个 T 对象。

   cpp

   ```cpp
   T t1;
   T t2 = t1;
   ```

3. **可复制赋值**（Copy Assignable）：可以将一个 T 对象的值赋给另一个 T 对象。

   cpp

   ```cpp
   T t1, t2;
   t1 = t2;
   ```

4. **可析构**（Destructible）：对象可以被销毁（通常由编译器自动保证）。

   cpp

   ```cpp
   { T t; } // 离开作用域时自动调用析构函数
   ```

5. **可交换**（Swappable）：可以通过 std::swap 交换两个 T 对象的值。

   cpp

   ```cpp
   T t1, t2;
   std::swap(t1, t2);
   ```

这些要求确保类型 T 具有基本的“值语义”（value semantics），可以像内置类型（如 int、double）一样被安全地复制、移动、销毁和交换。

与其他概念的关系

- **Regular 类型**：SemiRegular 是 Regular 的子集。Regular 类型在 SemiRegular 的基础上还要求**可比较**（Equality Comparable），即支持 == 和 != 操作。例如：

  cpp

  ```cpp
  bool b = (t1 == t2); // Regular 类型需要支持
  ```

  因此，int 是 Regular 类型，但一个没有定义 == 的类可能是 SemiRegular 而非 Regular。

- **MoveConstructible 和 MoveAssignable**：C++11 引入了移动语义，SemiRegular 类型通常也隐式支持移动构造和移动赋值（尽管严格定义中不强制要求）。在现代 C++ 中，许多 SemiRegular 类型也满足这些要求。

为什么重要？

- **标准库兼容性**：C++ 标准库的容器（如 std::vector<T>、std::map<K, V>）和算法通常要求类型至少是 SemiRegular。这是为了确保元素可以被构造、复制、赋值和销毁，而不会引发未定义行为。

- **通用编程**：在模板编程中，SemiRegular 提供了一个最低限度的类型约束，确保泛型代码可以安全操作未知类型。

- **C++20 Concepts**：C++20 引入了 std::semiregular 概念（在 <concepts> 中），用于在编译期检查类型是否满足 SemiRegular 要求：

  cpp

  ```cpp
  #include <concepts>
  
  template<typename T>
  requires std::semiregular<T>
  void use(T t) { /* ... */ }
  ```

示例

SemiRegular 类型

cpp

```cpp
#include <iostream>
#include <concepts>

struct MyType {
    int value;

    MyType() = default;                     // 默认构造
    MyType(const MyType&) = default;        // 复制构造
    MyType& operator=(const MyType&) = default; // 复制赋值
    // 析构函数自动提供
    // std::swap 自动支持（通过成员变量的 swap）
};

static_assert(std::semiregular<MyType>); // C++20 检查

int main() {
    MyType a, b;
    a.value = 42;
    b = a; // 复制赋值
    std::swap(a, b); // 交换
    std::cout << a.value << "\n"; // 输出: 42
}
```

MyType 是 SemiRegular，因为它满足默认构造、复制构造、复制赋值、析构和交换。

非 SemiRegular 类型

cpp

```cpp
struct NonCopyable {
    NonCopyable() = default;
    NonCopyable(const NonCopyable&) = delete; // 禁用复制构造
    NonCopyable& operator=(const NonCopyable&) = delete; // 禁用复制赋值
};

static_assert(!std::semiregular<NonCopyable>); // 不满足
```

NonCopyable 不是 SemiRegular，因为它缺少复制构造和赋值。

在你的代码中的关联

结合你之前的代码（涉及 std::enable_if、迭代器和 std::tuple），SemiRegular 概念与容器（如 std::vector）和类型约束相关：

- **std::vector<T>**：要求 T 是 SemiRegular（或更严格的 Regular），以确保元素可以被构造、复制和销毁。你的 Foo 类中使用了 std::vector<T>::iterator，而 std::vector 的元素类型必须满足这些要求。
- **std::tuple**：std::tuple<Ts...> 的元素类型通常需要是 SemiRegular，以支持复制和赋值操作。你的 t2（std::tuple<int, std::string_view, double>）满足这些要求，因为 int、std::string_view 和 double 都是 SemiRegular。

总结

- **SemiRegular** 描述了一种类型，具有默认构造、复制构造、复制赋值、析构和可交换的属性。
- 它是 Regular 类型（额外要求可比较）的子集，常用于标准库容器和算法。
- C++20 提供了 std::semiregular 概念来检查这些属性。
- 典型例子包括 int、std::string、自定义结构体（带默认/复制操作）；反例包括禁用复制的类型。
- 在容器（如 std::vector）和模板编程中，SemiRegular 是确保类型安全性和通用性的基础。

如果你有具体场景（例如某个类型是否 SemiRegular，或如何在你的代码中应用），可以提供更多细节，我会进一步分析！