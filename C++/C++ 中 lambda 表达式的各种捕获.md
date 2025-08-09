#  C++ 中 lambda 表达式的各种捕获

这段代码展示了 C++ 中 lambda 表达式的各种捕获方式，特别是与 this 和 *this 相关的捕获机制，以及它们如何影响对类成员和局部变量的访问。以下是对代码的逐步解释：

```c++
#include <cassert>

struct Outer {
    int x = 42; // member
    void fun() {
        int y = 7; // local variable

        [this]{ // capture this by reference
            this->x = 42; // OK
            // For the purposes of name lookup, the lambda operator() 
            // belongs to the local scope, i.e. 'this' is Outer.
            // y not captured
        }();
        [*this]{ // C++17 capture by copy
            assert(x == 42); // x is immutable
            // y is not captured
        }();
        [&]{ // implicitly capture this
            x = 42; // OK
            y = 7; // y captured by reference
        }();
        [&, this]{ // same as above, explicit spelling
            x = 42; // OK
            y = 7; //  y captured by reference
        }();
        [&, *this]{ // C++17 capture by copy
            assert(x == 42); // x is immutable
            y = 7; // y captured by reference
        }();
        [=]{ // deprecated in C++20, confusing implicit capture
            x = 42; // OK, we capture the 'this' pointer by value
                    // not the object itself
            assert(y == 7); // y is immutable
        }();
        [=, this]{ // explicit spelling remains valid
            x = 42;
            assert(y == 7); // y is immutable
        }();
        [=, *this]{ // everything by copy, C++17
            assert(x == 42); // x is immutable
            assert(y == 7); // y is immutable
        }();
    }
};

int main() {
    Outer o;
    o.fun();
}
```



------

背景知识

- **Lambda 表达式**：C++ 中的 lambda 是一种匿名函数，形式为 [捕获列表]{ 函数体 }()，其中捕获列表指定如何捕获外部作用域的变量。
- **this 的捕获**：
  - this 是一个指向当前对象（Outer 实例）的指针。
  - 捕获 this 允许 lambda 访问类的成员变量（如 x）。
- ***this 的捕获**（C++17 引入）：
  - 按值捕获整个对象（*this），而不是仅捕获指针。
- **捕获方式**：
  - 按引用（&）：捕获变量的引用，可修改原变量。
  - 按值（=, 或显式列出）：捕获变量的副本，默认不可修改（除非 lambda 声明为 mutable）。

------

代码分析

定义

cpp

```cpp
struct Outer {
    int x = 42; // 成员变量
    void fun() {
        int y = 7; // 局部变量
        // 以下是各种 lambda 捕获方式
    }
};
```

- **x** 是 Outer 的成员变量，初始值为 42。
- **y** 是 fun 函数中的局部变量，初始值为 7。
- 每个 lambda 立即执行（()），展示捕获效果。

------

1. 捕获 [this]

cpp

```cpp
[this]{ // 捕获 this 指针
    this->x = 42; // OK
    // y 未被捕获
}();
```

- **捕获**：按值捕获 this 指针。
- **行为**：
  - this->x 可以访问和修改 Outer 的成员 x，因为 this 是指向当前对象的指针。
  - y 未被捕获，无法访问（编译错误）。
- **命名解析**：lambda 的 operator() 被视为 fun 的局部作用域，但可以通过 this 访问类的成员。

------

2. 捕获 [*this]（C++17）

cpp

```cpp
[*this]{ // 按值捕获整个对象
    assert(x == 42); // x 是只读的
    // y 未被捕获
}();
```

- **捕获**：按值捕获 *this，即复制整个 Outer 对象。
- **行为**：
  - x 是副本的一部分，默认不可修改（lambda 未声明 mutable）。
  - assert(x == 42) 检查副本的 x 是否为 42。
  - y 未被捕获，无法访问。
- **特点**：*this 捕获的是对象副本，与原始对象独立。

------

3. 捕获 [&]

cpp

```cpp
[&]{ // 隐式按引用捕获所有变量
    x = 42; // OK
    y = 7; // y 按引用捕获
}();
```

- **捕获**：按引用捕获所有外部变量，包括 this 和 y。
- **行为**：
  - x = 42 修改原始对象的 x，因为 this 被隐式捕获为引用。
  - y = 7 修改局部变量 y，因为 y 被按引用捕获。
- **特点**：简单但可能捕获过多变量，增加意外修改的风险。

------

4. 捕获 [&, this]

cpp

```cpp
[&, this]{ // 同上，显式捕获 this
    x = 42; // OK
    y = 7; // y 按引用捕获
}();
```

- **捕获**：
  - &：按引用捕获所有局部变量（如 y）。
  - this：显式按值捕获 this 指针（但效果等同于引用，因为可以通过 this 修改对象）。
- **行为**：与 [&] 相同，x 和 y 都可以修改。
- **特点**：显式指定 this，更清晰地表达意图。

------

5. 捕获 [&, *this]（C++17）

cpp

```cpp
[&, *this]{ // 混合捕获
    assert(x == 42); // x 是只读的
    y = 7; // y 按引用捕获
}();
```

- **捕获**：
  - &：按引用捕获局部变量（如 y）。
  - *this：按值捕获整个对象。
- **行为**：
  - x 是对象副本的一部分，不可修改。
  - y 是按引用捕获的，可以修改原始局部变量。
- **特点**：混合捕获，x 只读，y 可写。

------

6. 捕获 [=]（C++20 中不推荐）

cpp

```cpp
[=]{ // 隐式按值捕获所有变量
    x = 42; // OK，捕获的是 this 指针
    assert(y == 7); // y 是只读的
}();
```

- **捕获**：按值捕获所有变量，包括 this 和 y。
- **行为**：
  - x = 42：捕获的是 this 指针（而非 *this），可以通过 this 修改原始对象的 x。
  - y 是副本，不可修改。
- **问题**：[=] 隐式捕获 this 的行为容易引起混淆，因此 C++20 标记为不推荐。

------

7. 捕获 [=, this]

cpp

```cpp
[=, this]{ // 显式捕获 this
    x = 42;
    assert(y == 7); // y 是只读的
}();
```

- **捕获**：
  - =：按值捕获局部变量（如 y）。
  - this：显式按值捕获 this 指针。
- **行为**：
  - x = 42：通过 this 修改原始对象的 x。
  - y 是副本，只读。
- **特点**：比 [=] 更清晰，C++20 后推荐这种写法。

------

8. 捕获 [=, *this]（C++17）

cpp

```cpp
[=, *this]{ // 所有变量按值捕获
    assert(x == 42); // x 是只读的
    assert(y == 7); // y 是只读的
}();
```

- **捕获**：
  - =：按值捕获局部变量（如 y）。
  - *this：按值捕获整个对象。
- **行为**：
  - x 和 y 都是副本，不可修改。
- **特点**：完全隔离的副本操作，适合不需要修改外部状态的场景。

------

总结表

| 捕获方式   | this 捕获    | x 可修改? | y 捕获 | y 可修改? |
| ---------- | ------------ | --------- | ------ | --------- |
| [this]     | 指针（值）   | 是        | 未捕获 | -         |
| [*this]    | 对象（值）   | 否        | 未捕获 | -         |
| [&]        | 指针（引用） | 是        | 引用   | 是        |
| [&, this]  | 指针（值）   | 是        | 引用   | 是        |
| [&, *this] | 对象（值）   | 否        | 引用   | 是        |
| [=]        | 指针（值）   | 是        | 值     | 否        |
| [=, this]  | 指针（值）   | 是        | 值     | 否        |
| [=, *this] | 对象（值）   | 否        | 值     | 否        |

------

关键点

1. **this vs \*this**：
   - this 捕获指针，可以修改原始对象。
   - *this 捕获对象副本，副本默认不可修改。
2. **C++20 的变化**：
   - [=] 隐式捕获 this 的行为不推荐，使用 [=, this] 替代。
3. **灵活性**：
   - 混合捕获（如 [&, *this]）允许对不同变量采用不同策略。
4. **性能**：
   - 按值捕获 *this 会复制整个对象，可能有开销。

这些捕获方式为开发者提供了细粒度的控制，适用于不同的场景（如性能优化、线程安全等）。