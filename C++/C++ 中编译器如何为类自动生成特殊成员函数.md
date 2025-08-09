# C++ 中编译器如何为类自动生成特殊成员函数

**这段代码展示了 C++ 中编译器如何为类自动生成特殊成员函数（构造、赋值、析构），**

**以及当类成员具有特定属性（如“仅移动”）时，这些自动生成的行为如何受到影响。**

以下是逐步解释。

“零规则”是从单一责任原则衍生出来的类设计原则。

除非类的唯一目的是管理所有权，否则类不应该定义任何特殊的成员函数。

代码:

```c++
#include <string>
#include <vector>

// Piecewise Copy/Move constructors, Copy/Move assignments 
// and destructor will be provided by the compiler.
struct MyClass {
    MyClass(const std::string& label, 
        const std::vector<int>& data) : label_(label), data_(data) {}
private:
    std::string label_;
    std::vector<int> data_;
};

struct MoveOnly {
    MoveOnly(MoveOnly&&) = default;
};

// If one of the members is move-only, the copy constructor
// and copy assignment will not be generated.
struct NoCopyGenerated {
    MoveOnly moveonly_;
};

int main() {
    static_assert(not std::is_copy_constructible_v<NoCopyGenerated>);
}
```

------

代码概览

- 定义了两个结构体 MyClass 和 NoCopyGenerated，以及一个辅助结构体 MoveOnly。
- 展示了编译器如何为 MyClass 生成默认的特殊成员函数。
- 展示了当成员是“仅移动”类型（如 MoveOnly）时，NoCopyGenerated 的拷贝构造函数和拷贝赋值运算符不会被生成。
- 使用 static_assert 检查 NoCopyGenerated 是否不可拷贝。

------

关键组件

1. **头文件**

cpp

```cpp
#include <string>
#include <vector>
```

- <string>：提供 std::string。
- <vector>：提供 std::vector<int>。
- **MyClass 定义**

cpp

```cpp
struct MyClass {
    MyClass(const std::string& label, 
        const std::vector<int>& data) : label_(label), data_(data) {}
private:
    std::string label_;
    std::vector<int> data_;
};
```

- **构造函数**：
  - 接受 const std::string& 和 const std::vector<int>& 参数，初始化成员 label_ 和 data_。
- **成员**：
  - label_：std::string 类型，支持拷贝和移动。
  - data_：std::vector<int> 类型，也支持拷贝和移动。
- **编译器生成的特殊成员函数**：
  - 因为没有显式定义以下函数，编译器会自动生成（按成员逐一操作，称为“piecewise”）：
    1. **拷贝构造函数**：MyClass(const MyClass&)，拷贝 label_ 和 data_。
    2. **移动构造函数**：MyClass(MyClass&&)，移动 label_ 和 data_。
    3. **拷贝赋值运算符**：MyClass& operator=(const MyClass&)，逐一赋值。
    4. **移动赋值运算符**：MyClass& operator=(MyClass&&)，逐一移动。
    5. **析构函数**：~MyClass()，销毁 label_ 和 data_。
- **条件**：
  - 这些函数能生成的前提是所有成员（std::string 和 std::vector<int>）都支持相应的操作。
  - 这里，std::string 和 std::vector<int> 都可拷贝和移动，因此所有特殊成员函数都会生成。
- **MoveOnly 定义**

cpp

```cpp
struct MoveOnly {
    MoveOnly(MoveOnly&&) = default;
};
```

- **MoveOnly**：
  - 显式定义了移动构造函数，并使用 = default 让编译器生成默认实现。
  - 没有定义拷贝构造函数或拷贝赋值运算符，因此它们被隐式删除（= delete）。
- **特性**：
  - MoveOnly 是“仅移动”类型：
    - 可移动（MoveOnly(MoveOnly&&) 可用）。
    - 不可拷贝（拷贝构造函数和赋值运算符不可用）。
- **NoCopyGenerated 定义**

cpp

```cpp
struct NoCopyGenerated {
    MoveOnly moveonly_;
};
```

- **成员**：
  - moveonly_：类型为 MoveOnly，是“仅移动”类型。
- **编译器行为**：
  - 因为没有显式定义特殊成员函数，编译器会尝试生成：
    1. **拷贝构造函数**：需要拷贝 moveonly_，但 MoveOnly 不可拷贝，因此不会生成。
    2. **移动构造函数**：需要移动 moveonly_，MoveOnly 支持移动，因此会生成。
    3. **拷贝赋值运算符**：需要拷贝赋值 moveonly_，不可行，因此不会生成。
    4. **移动赋值运算符**：需要移动赋值 moveonly_，可行，因此会生成。
    5. **析构函数**：MoveOnly 的析构函数可用，因此会生成。
- **结果**：
  - NoCopyGenerated 是“不可拷贝”的，因为其成员 moveonly_ 阻止了拷贝相关函数的生成。
- **main 函数**

cpp

```cpp
int main() {
    static_assert(not std::is_copy_constructible_v<NoCopyGenerated>);
}
```

- **std::is_copy_constructible_v**：
  - C++17 引入的类型特性，检查类型是否可拷贝构造。
  - 返回 true 如果拷贝构造函数可用，否则返回 false。
- **static_assert**：
  - 检查 NoCopyGenerated 是否不可拷贝构造。
  - not std::is_copy_constructible_v<NoCopyGenerated> 为 true，因为拷贝构造函数未生成。
  - 如果断言失败（即类型可拷贝），编译会报错。
- **结果**：
  - 编译通过，证明 NoCopyGenerated 不可拷贝。

------

为什么这样工作？

1. **编译器生成规则**：
   - C++ 编译器根据成员的属性逐一（piecewise）生成特殊成员函数。
   - 如果所有成员都支持某操作（如拷贝或移动），则生成相应的函数。
   - 如果任一成员不支持某操作，则该函数不会生成（隐式删除）。
2. **MyClass**：
   - std::string 和 std::vector<int> 都支持拷贝和移动，因此所有特殊成员函数都生成。
3. **NoCopyGenerated**：
   - MoveOnly 只支持移动，不支持拷贝，导致拷贝构造函数和拷贝赋值运算符无法生成。
4. **C++ 的移动语义**：
   - “仅移动”类型（如 MoveOnly）是现代 C++ 的重要特性，用于资源管理（如 std::unique_ptr）。

------

输出

- 没有运行时输出，因为代码只包含编译时检查。
- static_assert 通过，编译成功。

------

使用场景

- **MyClass**：
  - 表示可拷贝和移动的对象，适用于需要复制或转移数据的场景。
- **NoCopyGenerated**：
  - 表示不可拷贝的对象，适用于需要确保独占资源（如文件句柄、网络连接）的场景。
- **MoveOnly**：
  - 模拟类似 std::unique_ptr 的行为，强调所有权转移。

------

总结

- MyClass 展示了编译器如何为支持拷贝和移动的成员自动生成特殊成员函数。
- NoCopyGenerated 展示了当成员是“仅移动”类型时，拷贝相关函数不会生成。
- static_assert 验证了 NoCopyGenerated 的不可拷贝性。
- 这段代码突出 C++ 中特殊成员函数生成规则与移动语义的交互。

