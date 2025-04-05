# Explicit object parameter (a.k.a. deducing this)

这段代码展示了 C++ 中关于成员函数的几种声明方式，特别是与“显式对象参数”（explicit object parameter，C++23 引入的新特性）相关的语法演变和应用。以下是对代码的逐步解释：

```c++
#include <cstdio>
#include <utility>

// Old spelling
struct Old {
    // Also can be spelled as: "void method() {}"
    void method() & { puts("Mutable lvalue"); }
    // Also can be spelled as: "void method() const {}"
    void method() const& { puts("Immutable lvalue"); }
    void method() && { puts("Rvalue"); }
};

// New spelling
struct New {
    void method(this New&) { puts("Mutable lvalue"); }
    void method(this const New&) { puts("Immutable lvalue"); }
    void method(this New&&) { puts("Rvalue"); }
};

// One deduced method that handles all three cases
struct Universal {
    void method(this auto&& self) {
        using T = decltype(self);
        if constexpr (std::is_lvalue_reference_v<T>) {
            if constexpr (std::is_const_v<std::remove_reference_t<T>>)
                puts("Immutable lvalue");
            else
                puts("Mutable lvalue");
        } else if constexpr (std::is_rvalue_reference_v<T>)
            puts("Rvalue");
    }
};

// A practical example of the above
struct Practical {
    // Single getter that handles all three variants
    auto&& get(this auto&& self) {
        // Based on the type of this, forward as lvalue, or rvalue
        return std::forward_like<decltype(self)>(self.value);
    }
private:
    int value = 0;
};

// We can also spell a copy
struct Object {
    // Operate on a copy of the object
    void method(this Object self) {}
};

// The deduction path can also be used as CRTP replacement
struct InjectMethod {
    // Because self is deduced, it will be the static type 
    // at the call site, i.e. the derived type.
    void do_stuff(this const auto& self) {
        puts("I did stuff.");
        self.do_other_stuff();
    }
};

struct Derived : InjectMethod {
    void do_other_stuff() const {
        puts("And then other stuff.");
    }
};

int main() {
    Old a;
    a.method();
    std::as_const(a).method();
    Old{}.method();

    New b;
    b.method();
    std::as_const(b).method();
    New{}.method();

    Universal c;
    c.method();
    std::as_const(c).method();
    Universal{}.method();

    Practical d;
    static_assert(std::is_same_v<decltype(d.get()), int&>);
    static_assert(std::is_same_v<decltype(std::as_const(d).get()), const int&>);
    static_assert(std::is_same_v<decltype(Practical{}.get()), int&&>);

    Derived e;
    e.do_stuff();
}
```



------

1. **Old Spelling（传统写法）**

cpp

```cpp
struct Old {
    void method() & { /* lvalue */ }
    void method() const& { /* immutable lvalue */ }
    void method() && { /* rvalue */ }
};
```

- 这是 C++11 开始支持的传统写法，使用**引用限定符**（& 和 &&）来控制成员函数在不同对象类型上的行为：
  - void method() &：只对左值（lvalue，例如非临时对象）调用。
  - void method() const&：只对常量左值调用（不可修改的左值）。
  - void method() &&：只对右值（rvalue，例如临时对象）调用。
- 这种方式需要为每种情况分别定义函数，如果逻辑类似，会导致代码重复。

------

2. **New Spelling（显式对象参数写法）**

cpp

```cpp
struct New {
    void method(this New&) { /* lvalue */ }
    void method(this const New&) { /* immutable lvalue */ }
    void method(this New&&) { /* rvalue */ }
};
```

- 这是 C++23 引入的**显式对象参数**语法，将 this 指针作为函数的显式参数。
- 传统写法中，this 是隐式的，而这里通过参数类型明确指定调用时的对象类型：
  - this New&：左值对象。
  - this const New&：常量左值对象。
  - this New&&：右值对象。
- 优点是语法更直观，this 的类型直接出现在参数列表中，与普通函数参数一致。

------

3. **Universal（单一推导方法）**

cpp

```cpp
struct Universal {
    void method(this auto&& self) {
        // Will deduce one of: Universal&, const Universal& or Universal&&
    }
};
```

- 这里使用了 C++20 的 auto 结合 C++23 的显式对象参数，进一步简化代码。
- this auto&& self 表示 self 的类型会根据调用上下文自动推导为：
  - Universal&（左值）。
  - const Universal&（常量左值）。
  - Universal&&（右值）。
- 这种写法通过一个函数处理所有情况，避免了多次定义，类似于模板的类型推导。

------

4. **Practical（实际应用示例）**

cpp

```cpp
struct Practical {
    auto&& get(this auto&& self) {
        return std::forward_like<decltype(self)>(self.value);
    }
private:
    int value = 0;
};
```

- 这是一个实际应用，展示如何利用显式对象参数和 std::forward_like（C++23）实现通用 getter。
- auto&& 推导出 self 的类型（可能是 Practical&、const Practical& 或 Practical&&）。
- std::forward_like<decltype(self)>(self.value) 根据 self 的值类别（lvalue 或 rvalue）完美转发 value：
  - 如果 self 是左值，返回左值引用。
  - 如果 self 是右值，返回右值引用。
- 这允许单一函数处理所有调用场景，同时保持正确的引用语义。

------

5. **Object（基于副本操作）**

cpp

```cpp
struct Object {
    void method(this Object self) {}
};
```

- 这里 this Object self 表示按值传递对象，即调用时会创建对象的副本。
- 这种写法适用于需要操作对象副本的场景（例如修改副本而不影响原始对象）。
- 注意：由于是值传递，可能会有性能开销（拷贝构造），适合小型对象或明确需要副本的情况。

------

6. **InjectMethod（CRTP 替代方案）**

cpp

```cpp
struct InjectMethod {
    void do_stuff(this const auto& self) {
        puts("I did stuff.");
        self.do_other_stuff();
    }
};

struct Derived : InjectMethod {
    void do_other_stuff() const {
        puts("And then other stuff.");
    }
};
```

- 这展示了一种将显式对象参数用作 **CRTP（Curiously Recurring Template Pattern）** 替代方案的方法。
- this const auto& self 表示 self 是调用者的类型（这里是 Derived 的常量引用），通过类型推导自动绑定到派生类。
- 在 do_stuff 中，self.do_other_stuff() 调用的是 Derived 中的 do_other_stuff，实现了类似 CRTP 的静态多态。
- 与传统 CRTP（需要模板参数）相比，这种写法更简洁，避免了显式模板声明。

------

使用示例

cpp

```cpp
int main() {
    Practical p;
    const Practical cp;
    p.get();              // 返回 int&（左值）
    cp.get();             // 返回 const int&（常量左值）
    Practical().get();    // 返回 int&&（右值）

    Derived d;
    d.do_stuff();         // 输出：
                          // I did stuff.
                          // And then other stuff.
    return 0;
}
```

------

总结

这段代码展示了 C++ 从传统引用限定符到 C++23 显式对象参数的演进：

1. **传统写法**：需要为每种值类别单独定义函数。
2. **显式对象参数**：更直观地将 this 作为参数，语法更统一。
3. **结合 auto 和转发**：通过类型推导和 std::forward_like，可以用单一函数优雅地处理所有情况。
4. **替代模式**：如基于副本操作或替代 CRTP 的新用法。

这些特性提高了代码的灵活性和简洁性，尤其在需要处理多种值类别或实现静态多态时非常有用。