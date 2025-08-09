

# C++ Returning covariant return types from overriding virtual functions

When working with virtual functions, we can encounter situations where we need to return a different type from an overriding function.

While this isn’t possible in the general case, the types returned are allowed to be different if they are covariant.

Types returned by *Base::fun()* and *Derived::fun()* are covariant if:

- both return a pointer or reference
- the return type of *Base::fun()* is an unambiguous accessible base class of the type returned by *Derived::fun()*
- the type returned by *Derived::fun()* isn’t more cv-qualified

The prototypical use for covariant return types is a *clone()* function.

```C++
#include <memory>
#include <iostream>

struct Base {
    virtual Base* clone() const {
        std::cout << "Base::clone()\n";
        return new Base(*this);
    }
    virtual ~Base() = default;
};

struct Derived : Base {
    Derived* clone() const override {
        std::cout << "Derived::clone()\n";
        return new Derived(*this);
    }
    ~Derived() override = default;
};

// Because covariant types require a raw reference or pointer, 
// we can't make smart pointers covariant
struct BaseSmart {
    virtual ~BaseSmart() = default;

    // A clone wrapper for the base type.
    // If you also want std::unique_ptr<DerivedSmart> the solutions get lot more complicated.
    friend std::unique_ptr<BaseSmart> clone(const std::unique_ptr<BaseSmart>& src) {
        return std::unique_ptr<BaseSmart>(src->clone_impl());
    }
private:
    virtual BaseSmart* clone_impl() const {
        std::cout << "BaseSmart::clone_impl()\n";
        return new BaseSmart(*this);
    }
};

struct DerivedSmart : BaseSmart {
    ~DerivedSmart() override = default;
private:
    DerivedSmart* clone_impl() const override {
        std::cout << "DerivedSmart::clone_impl()\n";
        return new DerivedSmart(*this);
    }
};

Base* get_object() { return new Derived(); }
std::unique_ptr<BaseSmart> get_smart() { return std::unique_ptr<BaseSmart>{new DerivedSmart()}; }

int main() {
    Base* p1 = get_object();
    Base* p2 = p1->clone(); // Calls Derived::clone()
    delete p1;
    delete p2;

    std::unique_ptr<BaseSmart> p3 = get_smart();
    std::unique_ptr<BaseSmart> p4 = clone(p3); // Calls DerivedSmart::clone_impl
}
```

这段代码展示了 C++ 中使用虚函数实现克隆（clone）模式，并对比了原始指针和智能指针（std::unique_ptr）在多态场景下的用法。

代码通过两个类层次结构（Base/Derived 和 BaseSmart/DerivedSmart）演示了如何在基类中定义克隆接口，以及智能指针如何通过辅助函数实现类似功能。

以下是对代码的详细解释：

------

**代码结构**

1. **头文件**：
   - <memory>：提供 std::unique_ptr。
   - <iostream>：提供 std::cout。
2. **主要内容**：
   - Base 和 Derived：使用原始指针实现协变返回类型（covariant return type）的克隆。
   - BaseSmart 和 DerivedSmart：使用 std::unique_ptr 实现克隆。
   - main：测试两种方式。

------

**代码逐步解释**

**1. Base 和 Derived（原始指针）**

cpp

```cpp
struct Base {
    virtual Base* clone() const {
        std::cout << "Base::clone()\n";
        return new Base(*this);
    }
    virtual ~Base() = default;
};

struct Derived : Base {
    Derived* clone() const override {
        std::cout << "Derived::clone()\n";
        return new Derived(*this);
    }
    ~Derived() override = default;
};
```

- **Base**：
  - virtual Base* clone() const：虚函数，返回指向新对象的指针，用于克隆。
  - 默认实现：创建 Base 副本。
  - virtual ~Base()：虚析构函数，确保多态删除安全。
- **Derived**：
  - Derived* clone() const override：覆盖基类函数，返回 Derived*。
  - **协变返回类型**（covariant return type）：C++ 允许派生类返回更具体的类型（Derived* 是 Base* 的子类型）。
- **行为**：
  - 调用时根据动态类型选择实现。

cpp

```cpp
Base* get_object() { return new Derived(); }
Base* p1 = get_object();
Base* p2 = p1->clone(); // Calls Derived::clone()
delete p1;
delete p2;
```

- **get_object()**：
  - 返回 Derived 对象的 Base* 指针。
- **p1->clone()**：
  - p1 的动态类型是 Derived，调用 Derived::clone()。
  - 输出：Derived::clone()。
  - 返回 Derived*，赋值给 Base* p2。
- **清理**：
  - 手动 delete 释放内存。

------

**2. BaseSmart 和 DerivedSmart（智能指针）**

cpp

```cpp
struct BaseSmart {
    virtual ~BaseSmart() = default;
    friend std::unique_ptr<BaseSmart> clone(const std::unique_ptr<BaseSmart>& src) {
        return std::unique_ptr<BaseSmart>(src->clone_impl());
    }
private:
    virtual BaseSmart* clone_impl() const {
        std::cout << "BaseSmart::clone_impl()\n";
        return new BaseSmart(*this);
    }
};

struct DerivedSmart : BaseSmart {
    ~DerivedSmart() override = default;
private:
    DerivedSmart* clone_impl() const override {
        std::cout << "DerivedSmart::clone_impl()\n";
        return new DerivedSmart(*this);
    }
};
```

- **BaseSmart**：
  - virtual BaseSmart* clone_impl() const：私有虚函数，返回原始指针。
  - friend std::unique_ptr<BaseSmart> clone(...)：
    - 辅助函数，接受 std::unique_ptr<BaseSmart>，调用 clone_impl() 并包装结果。
    - 返回类型固定为 std::unique_ptr<BaseSmart>，无法协变。
- **DerivedSmart**：
  - 覆盖 clone_impl()，返回 DerivedSmart*（协变）。
- **限制**：
  - std::unique_ptr 不支持协变返回类型，因此通过 friend 函数统一返回基类智能指针。

cpp

```cpp
std::unique_ptr<BaseSmart> get_smart() { return std::unique_ptr<BaseSmart>{new DerivedSmart()}; }
std::unique_ptr<BaseSmart> p3 = get_smart();
std::unique_ptr<BaseSmart> p4 = clone(p3); // Calls DerivedSmart::clone_impl
```

- **get_smart()**：
  - 返回 DerivedSmart 对象的 std::unique_ptr<BaseSmart>。
- **clone(p3)**：
  - p3 的动态类型是 DerivedSmart，调用 DerivedSmart::clone_impl()。
  - 输出：DerivedSmart::clone_impl()。
  - 返回的 DerivedSmart* 被包装为 std::unique_ptr<BaseSmart>。
- **清理**：
  - 智能指针自动释放，无需手动 delete。

------

**关键技术点**

1. **协变返回类型**：
   - 原始指针支持（如 Base* 到 Derived*）。
   - 智能指针（如 std::unique_ptr）不支持，因为模板类型是固定的。
2. **虚函数**：
   - 多态克隆基于动态类型。
3. **智能指针克隆**：
   - 使用私有虚函数和外部 friend 函数绕过协变限制。
4. **RAII**：
   - std::unique_ptr 确保资源安全释放。

------

**输出总结**

```text
Derived::clone()
DerivedSmart::clone_impl()
```

------

**可能的改进或注意事项**

1. **复杂智能指针克隆**：
   - 若需 std::unique_ptr<DerivedSmart>，可使用模板工厂或类型擦除。
2. **异常安全**：
   - new 可能抛异常，clone 可添加 try。
3. **输出验证**：
   - 可添加成员变量验证克隆内容。

------

**总结**

- **Base/Derived**：展示原始指针的协变克隆。
- **BaseSmart/DerivedSmart**：展示智能指针的克隆实现。
- **用途**：多态对象的复制管理。
- **限制**：智能指针需额外设计绕过协变问题。

如果你有具体问题（例如实现 std::unique_ptr<DerivedSmart> 的克隆），欢迎提问！