

# C++ virtual and override keywords

Calling member functions based on the actual type of the object using the virtual and override keywords.

Virtual member functions modify the behaviour of calling member functions.

During object construction, the final overrider of each virtual member function is determined and stored in a table (vtable).

If the name lookup finds a virtual member function, it will insert an indirection through the vtable instead of calling it directly.

This ensures that the member function called will depend on the actual type of the object, not on the static base type listed in the code. This property is maintained even if the caller doesn’t have any information about the actual type of the object.



```C++
#include <iostream>

struct Base {
    virtual void fun() const { std::cout << "Base::fun()\n"; }
    virtual ~Base() = default; // If a class has at least one virtual
                               // member function, it should have 
                               // a virtual (or private) destructor
};

struct Derived : Base {
    void fun() const override { std::cout << "Derived::fun()\n"; }
    // void fun() override {} // Wouldn't compile, cv-qualifiers do not match
    ~Derived() override = default;
};

struct Abstract {
  	// Pure virtual member function, the derived classes must provide
    // an overriding implementation
    virtual void fun() const = 0;
    virtual ~Abstract() = default;
};

// Despite the method being 
void Abstract::fun() const { std::cout << "Abstract::fun()\n"; }

struct Concrete : Abstract {
    void fun() const override { std::cout << "Concrete::fun()\n"; }
    ~Concrete() override = default;
};

struct Middle : Base {
    void fun(int) const { std::cout << "Middle::fun(int)\n"; }
    ~Middle() override = default;
};

struct Final : Middle {
    void fun() const override { std::cout << "Final::fun()\n"; }
    ~Final() override = default;
};

int main() {
    Base base;
    base.fun(); // Calls Base::fun()

    Derived derived;
    derived.fun();       // Calls Derived::fun()
    derived.Base::fun(); // Calls Base::fun() explicitly

    Base& ref = derived;
    ref.fun(); // fun is virtual, actual type is Derived, 
               // final overrider is Derived::fun()
    ref.Base::fun(); // Calls Base::fun() explicitly

    std::cout << "--\n";

    // Abstract abstract; // Wouldn't compile, abstract class

    Concrete concrete;
    concrete.fun();
    concrete.Abstract::fun();

    Abstract* aref = &concrete;
    aref->fun();

    std::cout << "--\n";

    Middle mid;
    // mid.fun(); // Wouldn't compile
    mid.fun(42); // Calls Middle::fun(int)

    const Base& fin = Final{};
    fin.fun(); // Calls Final::fun()

    const Middle& hmm = Final{};
    // hmm.fun(); // Wouldn't compile, Base::fun() is hidden
    hmm.Base::fun(); // Calls Base::fun() explicitly
    static_cast<const Base&>(hmm).fun(); // Calls Final::fun()
}
```

这段代码展示了 C++ 中的虚函数（virtual functions）、抽象类（abstract classes）、函数覆盖（override）、隐藏（hiding）以及多态行为。代码通过多个类层次结构（Base、Derived、Abstract、Concrete、Middle、Final）演示了这些概念。以下是对代码的详细解释：

------

**代码结构**

1. **类定义**：
   - Base 和 Derived：基本的虚函数和覆盖。
   - Abstract 和 Concrete：抽象类和实现。
   - Middle 和 Final：隐藏和多态。
2. **头文件**：
   - <iostream>：用于 std::cout 输出。

------

**代码逐步解释**

**1. Base 和 Derived**

cpp

```cpp
struct Base {
    virtual void fun() const { std::cout << "Base::fun()\n"; }
    virtual ~Base() = default;
};

struct Derived : Base {
    void fun() const override { std::cout << "Derived::fun()\n"; }
    ~Derived() override = default;
};
```

- **Base**：
  - virtual void fun() const：虚函数，允许子类覆盖，const 表示不修改对象。
  - virtual ~Base()：虚析构函数，确保通过基类指针删除派生类对象时正确调用派生类的析构函数。
- **Derived**：
  - fun() const override：覆盖 Base::fun()，override 确保匹配虚函数签名。
  - 注释说明：如果去掉 const（如 void fun() override），编译失败，因为签名不匹配。

cpp

```cpp
Base base;
base.fun(); // Calls Base::fun()

Derived derived;
derived.fun();       // Calls Derived::fun()
derived.Base::fun(); // Calls Base::fun() explicitly
```

- **base.fun()**：
  - 调用 Base::fun()，输出 Base::fun()。
- **derived.fun()**：
  - 调用 Derived::fun()，输出 Derived::fun()。
- **derived.Base::fun()**：
  - 显式调用基类版本，输出 Base::fun()。

cpp

```cpp
Base& ref = derived;
ref.fun(); // Calls Derived::fun()
ref.Base::fun(); // Calls Base::fun()
```

- **ref**：
  - Base& 引用 Derived 对象，静态类型是 Base，动态类型是 Derived。
- **ref.fun()**：
  - fun 是虚函数，根据动态类型调用 Derived::fun()。
- **ref.Base::fun()**：
  - 显式调用基类版本，忽略虚函数机制。

------

**2. Abstract 和 Concrete**

cpp

```cpp
struct Abstract {
    virtual void fun() const = 0;
    virtual ~Abstract() = default;
};
void Abstract::fun() const { std::cout << "Abstract::fun()\n"; }

struct Concrete : Abstract {
    void fun() const override { std::cout << "Concrete::fun()\n"; }
    ~Concrete() override = default;
};
```

- **Abstract**：
  - virtual void fun() const = 0：纯虚函数，使 Abstract 成为抽象类，不能实例化。
  - 提供默认实现：尽管是纯虚函数，仍可定义实现，子类可选择调用。
- **Concrete**：
  - 覆盖 fun()，提供具体实现。

cpp

```cpp
// Abstract abstract; // 不合法，抽象类无法实例化

Concrete concrete;
concrete.fun();           // Calls Concrete::fun()
concrete.Abstract::fun(); // Calls Abstract::fun()

Abstract* aref = &concrete;
aref->fun();              // Calls Concrete::fun()
```

- **concrete.fun()**：
  - 调用 Concrete::fun()，输出 Concrete::fun()。
- **concrete.Abstract::fun()**：
  - 显式调用抽象基类的默认实现，输出 Abstract::fun()。
- **aref->fun()**：
  - aref 是 Abstract*，指向 Concrete 对象，虚函数调用 Concrete::fun()。

------

**3. Middle 和 Final**

cpp

```cpp
struct Middle : Base {
    void fun(int) const { std::cout << "Middle::fun(int)\n"; }
    ~Middle() override = default;
};

struct Final : Middle {
    void fun() const override { std::cout << "Final::fun()\n"; }
    ~Final() override = default;
};
```

- **Middle**：
  - 定义 fun(int) const，不覆盖 Base::fun()，而是隐藏它。
- **Final**：
  - 覆盖 Base::fun()，而非 Middle::fun(int)。

cpp

```cpp
Middle mid;
// mid.fun(); // 不合法
mid.fun(42); // Calls Middle::fun(int)
```

- **mid.fun()**：
  - 编译失败，Middle 隐藏了 Base::fun()，且无无参 fun()。
- **mid.fun(42)**：
  - 调用 Middle::fun(int)，输出 Middle::fun(int)。

cpp

```cpp
const Base& fin = Final{};
fin.fun(); // Calls Final::fun()
```

- **fin**：
  - Base& 引用 Final 对象。
- **fin.fun()**：
  - 虚函数调用，动态类型是 Final，输出 Final::fun()。

cpp

```cpp
const Middle& hmm = Final{};
// hmm.fun(); // 不合法
hmm.Base::fun(); // Calls Base::fun()
static_cast<const Base&>(hmm).fun(); // Calls Final::fun()
```

- **hmm.fun()**：
  - 编译失败，Middle 隐藏了 Base::fun()，且无匹配签名。
- **hmm.Base::fun()**：
  - 显式调用 Base::fun()，输出 Base::fun()。
- **static_cast**：
  - 转换为 Base&，虚函数调用 Final::fun()，输出 Final::fun()。

------

**关键技术点**

1. **虚函数**：
   - 通过动态类型决定调用，需 virtual 关键字。
2. **抽象类**：
   - 含纯虚函数（= 0），可提供默认实现。
3. **隐藏**：
   - 派生类定义同名函数隐藏基类版本。
4. **显式调用**：
   - 使用 Class::func() 指定版本。
5. **虚析构**：
   - 确保多态删除安全。

------

**输出总结**

```text
Base::fun()
Derived::fun()
Base::fun()
Derived::fun()
Base::fun()
--
Concrete::fun()
Abstract::fun()
Concrete::fun()
--
Middle::fun(int)
Final::fun()
Base::fun()
Final::fun()
```

------

**可能的改进或注意事项**

1. **错误检查**：
   - 可添加注释说明隐藏的后果。
2. **虚函数表**：
   - 可讨论 vtable 如何实现多态。
3. **签名匹配**：
   - 强调 override 的签名检查。

------

**总结**

- **虚函数**：实现动态多态。
- **隐藏**：影响静态调用。
- **抽象类**：强制子类实现。
- **用法**：展示继承和多态的典型场景。

如果你有具体问题（例如隐藏规则的细节），欢迎提问！