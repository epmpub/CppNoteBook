# 强制要求结构体字段必须显式初始化

Handling required fields with designated initializers

```c++
#include <string>
#include <iostream>
#include <iomanip>

template <typename T>
struct RequiredField {
    RequiredField(const T& value) : value(value) {}
    operator T&() { return value; }
    operator const T&() const { return value; }
    T value;
};

struct Options {
    RequiredField<int> id;
    std::string label = "default label";
};

int main() {
    Options options = { .id = 20 };
    // options.id == 20, options.label == "default label"

    std::cout << ".id == " << options.id << ", .label == " << std::quoted(options.label) << "\n";

    // Options opt = {.label = "some label"}; // Won't compile.
    // RequiredField is not constructible from empty initializer list.
}
```



------

代码概览

这段 C++ 代码定义了一个 RequiredField 模板结构体，用于强制要求某些结构体（如 Options）中的字段必须显式初始化，同时允许其他字段拥有默认值。它使用了现代 C++ 特性，例如指定初始化（C++20）和转换运算符。

------

关键组件

1. **RequiredField 模板结构体**

cpp

```cpp
template <typename T>
struct RequiredField {
    RequiredField(const T& value) : value(value) {}
    operator T&() { return value; }
    operator const T&() const { return value; }
    T value;
};
```

- **目的**：这是一个围绕类型 T 的值的包装器，确保该字段在初始化时必须显式提供。
- **构造函数**：RequiredField(const T& value) 接受一个值并将其存储在 value 成员中。
- **转换运算符**：
  - operator T&() 允许 RequiredField 对象隐式转换为类型 T 的可变引用。
  - operator const T&() const 允许它隐式转换为类型 T 的常量引用。
  - 这些运算符使得访问 value 时无需显式写 .value。
- **效果**：这让 RequiredField<T> 在大多数情况下表现得像一个 T，同时要求显式初始化。
- **Options 结构体**

cpp

```cpp
struct Options {
    RequiredField<int> id;
    std::string label = "default label";
};
```

- **字段**：
  - id 是一个 RequiredField<int>，意味着创建 Options 对象时必须显式初始化它。
  - label 是一个 std::string，默认值为 "default label"，因此它是可选的。
- **行为**：由于 id 是 RequiredField，你无法在不指定 id 的情况下创建 Options 对象。但 label 如果未提供，则使用默认值。
- **main 函数**

cpp

```cpp
int main() {
    Options options = { .id = 20 };
    // options.id == 20, options.label == "default label"

    std::cout << ".id == " << options.id << ", .label == " << std::quoted(options.label) << "\n";

    // Options opt = {.label = "some label"}; // 无法编译。
    // RequiredField 无法从空的初始化列表构造。
}
```

- **初始化**：
  - Options options = { .id = 20 }; 使用指定初始化（C++20）将 id 设置为 20。
  - 由于未指定 label，它采用默认值 "default label"。
  - 语法 .id = 20 显式初始化 id 成员，调用 RequiredField<int> 的构造函数并传入 20。
- **输出**：
  - std::cout << options.id 使用转换运算符 operator const int&() 访问 id.value（值为 20）。
  - std::quoted(options.label) 打印 "default label" 并带上引号以提高可读性。
  - 输出：.id == 20, .label == "default label"
- **注释掉的代码**：
  - Options opt = {.label = "some label"}; 无法编译，因为 id（一个 RequiredField<int>）未被初始化。
  - RequiredField 没有默认构造函数，也无法从空值初始化，因此强制要求提供 id。

------

为什么会这样工作

1. **强制要求字段**：
   - RequiredField 确保像 id 这样的字段不会意外地未初始化或默认构造。
   - 没有默认构造函数（例如 RequiredField() {}），编译器会强制你为 id 提供值。
2. **可选字段的灵活性**：
   - label 有默认值，因此在初始化时可以省略。
3. **转换运算符**：
   - 这些运算符使 RequiredField 的使用变得透明。你可以直接将 options.id 当作 int 使用（例如在 std::cout 中）。
4. **指定初始化**：
   - .id = 20 语法是 C++20 的特性，允许在初始化时指定结构体成员的名称，提高可读性。

------

如果尝试编译注释掉的代码会怎样？

cpp

```cpp
Options opt = {.label = "some label"};
```

- 这会失败，因为初始化列表中未提及 id（一个 RequiredField<int>）。
- 编译器会尝试默认构造 id，但 RequiredField 没有默认构造函数，只有接受 const T& 的构造函数。
- 结果：编译错误（例如“没有匹配的构造函数来初始化 RequiredField<int>”）。

------

实际应用场景

这种模式在设计 API 或数据结构时很有用，例如：

- 配置对象，其中 id 必须始终提供，但 label 可以有默认值。
- C++ 中的表单验证逻辑，要求某些输入不可跳过。

------

总结

- RequiredField 强制关键字段的显式初始化。
- Options 结合了必需字段（id）和可选字段（label）。
- 该代码展示了在现代 C++ 中使用指定初始化和转换运算符实现这一目标的简洁方法。
- 输出：.id == 20, .label == "default label"。
- 尝试跳过 id 会导致编译错误，正如设计初衷。