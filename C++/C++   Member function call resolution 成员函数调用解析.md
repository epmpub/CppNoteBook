# C++   Member function call resolution 成员函数调用解析

调用成员函数时，名称查找可能会导致令人惊讶的行为。

名称查找分步进行，当遇到任何匹配时将停止；这包括不可行的匹配。

查找始终从与变量的静态类型匹配的类类型范围开始，而不是从对象的动态类型开始。



```C++
#include <iostream>

struct Base {
    void fn() { std::cout << "Base::fn()\n"; }
    void fn(int) { std::cout << "Base::fn(int)\n"; }
    void call() { fn(); }
};

struct Derived : Base {
    void fn() { std::cout << "Derived::fn\n"; }
};

template <typename T>
struct MixinA {
    void fn() { std::cout << "MixinA::fn()\n"; }
    void call() {
        static_cast<T*>(this)->fn();
    }
};

struct MixinB {
    void fn() { std::cout << "MixinB::fn()\n"; }
    void call(this auto& self) {
        self.fn();
    }
};

struct UseA : MixinA<UseA> {
    void fn() { std::cout << "UseA::fn()\n"; }
};

struct UseB : MixinB {
    void fn() { std::cout << "UseB::fn()\n"; }
};

struct SideNote {
    void fn() {}
};

struct DerivedNote : SideNote {
    int fn;
};
 
int main() {
    Base base;
    base.fn();   // Calls Base::fn()
    base.fn(42); // Calls Base::fn(int)

    std::cout << "--\n";

    Derived derived;
    derived.fn(); // Calls Derived::fn()
    // derived.fn(42); // Will not compile, lookup finds Derived::fn()
                    // which isn't viable
    derived.call(); // Calls Base::call() -> Base::fn()
                    // 'this' in Base::call() is Base*

    std::cout << "--\n";

    // The lookup is based on the type of the variable,
    // not the dynamic type of the object
    Base& ref = derived;
    ref.fn(42); // Calls Base::fn(int)

    Base* ptr = &derived;
    ptr->fn(); // Calls Base::fn()

    // Same logic with member function pointers
    void (Base::*fn_ptr)(int) = &Base::fn;
    (derived.*fn_ptr)(42); // Calls Base::fn(int)

    std::cout << "--\n";

    UseA a;
    a.call(); // Calls MixinA::call() -> UseA::fn()

    UseB b;
    b.call(); // Calls MixinB::call() -> UseB::fn()

    // However, we are still dealing with the static type
    MixinB& mb = b;
    mb.call(); // Calls MixinB::call() -> MixinB::fn()
}
```



这段代码展示了 C++ 中继承、函数查找（name lookup）、成员函数指针以及调用派生类实现的不同方法的机制。它涉及基类和派生类的函数调用规则、静态类型与动态类型的区别，以及如何通过虚函数、CRTP（Curiously Recurring Template Pattern）和 C++23 的显式对象参数（deducing this）实现多态行为。以下是对代码的详细解释：

------

**代码结构**

1. **基本继承**：
   - Base 和 Derived，展示函数覆盖和查找规则。
2. **静态类型与动态类型**：
   - 使用引用和指针测试调用行为。
3. **成员函数指针**：
   - 展示如何绑定基类函数。
4. **调用派生类实现**：
   - 使用 CRTP（MixinA）和显式对象参数（MixinB）。

------

**代码逐步解释**

**1. 基本继承和函数查找**

cpp

```cpp
struct Base {
    void fn() {}
    void fn(int) {}
    void call() { fn(); }
};

struct Derived : Base {
    void fn() {}
};

Base base;
base.fn();   // Calls Base::fn()
base.fn(42); // Calls Base::fn(int)
```

- **Base**：
  - 定义两个重载函数 fn() 和 fn(int)，以及调用 fn() 的 call()。
- **base.fn() 和 base.fn(42)**：
  - base 是 Base 类型，调用对应的 Base 成员函数。
  - 查找基于静态类型（Base）。

cpp

```cpp
Derived derived;
derived.fn(); // Calls Derived::fn()
// derived.fn(42); // Will not compile
derived.call(); // Calls Base::call() -> Base::fn()
```

- **Derived**：
  - 定义 fn()，覆盖 Base::fn()。
- **derived.fn()**：
  - 调用 Derived::fn()，因为 Derived 的 fn() 隐藏了 Base 的所有 fn 重载。
- **derived.fn(42)**：
  - 不合法，Derived 只定义了无参 fn()，隐藏了 Base::fn(int)。
  - C++ 的名称查找规则：一旦在当前作用域找到名字，后续重载不考虑。
- **derived.call()**：
  - 调用继承的 Base::call()。
  - call() 中 this 是 Base* 类型，调用 Base::fn()。

------

**2. 静态类型决定调用**

cpp

```cpp
Base& ref = derived;
ref.fn(42); // Calls Base::fn(int)

Base* ptr = &derived;
ptr->fn(); // Calls Base::fn()
```

- **ref 和 ptr**：
  - ref 是 Base&，ptr 是 Base*，静态类型为 Base，动态类型为 Derived。
- **ref.fn(42)**：
  - 查找基于静态类型 Base，调用 Base::fn(int)。
- **ptr->fn()**：
  - 同理，调用 Base::fn()。
- **结论**：
  - 函数查找基于静态类型，与动态类型无关（非虚函数）。

------

**3. 成员函数指针**

cpp

```cpp
void (Base::*fn_ptr)(int) = &Base::fn;
(derived.*fn_ptr)(42); // Calls Base::fn(int)
```

- **fn_ptr**：
  - 成员函数指针，类型为 void (Base::*)(int)，绑定到 Base::fn(int)。
- **(derived.\*fn_ptr)(42)**：
  - 对 Derived 对象应用指针，调用 Base::fn(int)。
  - 成员函数指针锁定基类实现，不考虑派生类覆盖。

------

**4. 调用派生类实现：CRTP**

cpp

```cpp
template <typename T>
struct MixinA {
    void fn() {}
    void call() {
        static_cast<T*>(this)->fn();
    }
};

struct UseA : MixinA<UseA> {
    void fn() {}
};

UseA a;
a.call(); // Calls MixinA::call() -> UseA::fn()
```

- **MixinA**：
  - CRTP 模式，模板参数 T 是派生类。
  - call() 通过 static_cast<T*>(this) 将 this 转换为派生类类型，调用其 fn()。
- **UseA**：
  - 继承 MixinA<UseA>，提供自己的 fn()。
- **a.call()**：
  - 调用 MixinA::call()，this 被转换为 UseA*，执行 UseA::fn()。
- **优点**：
  - 静态多态，无虚函数开销。

------

**5. 调用派生类实现：显式对象参数**

cpp

```cpp
struct MixinB {
    void fn() {}
    void call(this auto& self) {
        self.fn();
    }
};

struct UseB : MixinB {
    void fn() {}
};

UseB b;
b.call(); // Calls MixinB::call() -> UseB::fn()
```

- **MixinB**：
  - 使用 C++23 的显式对象参数（this auto& self）。
  - call() 接受实际对象类型，调用其 fn()。
- **UseB**：
  - 继承 MixinB，提供自己的 fn()。
- **b.call()**：
  - self 是 UseB&，调用 UseB::fn()。
- **优点**：
  - 类型推导更灵活，避免显式模板参数。

------

**6. 静态类型限制**

cpp

```cpp
MixinB& mb = b;
mb.call(); // Calls MixinB::call() -> MixinB::fn()
```

- **mb**：
  - MixinB& 类型，引用 UseB 对象。
- **mb.call()**：
  - 静态类型是 MixinB，self 是 MixinB&，调用 MixinB::fn()。
- **结论**：
  - 即使动态类型是 UseB，调用仍基于静态类型。

------

**关键技术点**

1. **名称查找**：
   - 派生类函数隐藏基类所有同名函数。
2. **静态 vs 动态类型**：
   - 非虚函数调用取决于静态类型。
3. **CRTP**：
   - 通过模板实现静态多态。
4. **显式对象参数**：
   - C++23 新特性，基于调用者类型推导。
5. **虚函数替代**：
   - 未使用虚函数，通过其他机制实现多态。

------

**可能的改进或注意事项**

1. **虚函数对比**：
   - 可添加虚函数版本对比动态多态。
2. **输出验证**：
   - 当前无输出，可在 fn() 中添加打印。
3. **错误处理**：
   - CRTP 的 static_cast 需确保类型安全。

------

**总结**

- **基本继承**：展示隐藏和静态调用规则。
- **类型影响**：静态类型决定函数选择。
- **多态替代**：CRTP 和显式对象参数实现派生类调用。
- **用途**：理解 C++ 函数解析和多态机制。

如果你有具体问题（例如 CRTP 的其他应用），欢迎提问！