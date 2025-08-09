# Partial template specialization "模板部分特化"

```C++
template <typename T, typename U>
struct Type {};

// The specialization has to specialize something
template <typename X>
struct Type<int, X> {};

/* Wouldn't compile, doesn't specialize
template <typename X, typename Y>
struct Type<X,Y> {};
*/

/* Default arguments in specializations are not permitted
template <typename X = int>
struct Type<X, X> {};
*/

template <typename Type, Type Value>
struct NonType {};

// Dependent arguments cannot be freely specialized
template <>
struct NonType<int, 4> {}; // OK

/* Wouldn't compile
template <typename X>
struct NonType<X, 4> {};
*/

// The most specific specialization will be selected
template <typename T, typename U>
struct Ordering { static constexpr int v = 0; };

template <typename T, typename U>
struct Ordering<T*, U*> { static constexpr int v = 1; };

template <>
struct Ordering<int*, double*> { static constexpr int v = 2; };

#include <cstdint>

// Also works for variable templates
template <typename X, typename Y>
constexpr inline std::size_t ID = 0;

template <typename Y>
constexpr inline std::size_t ID<Y, int> = 10;

int main() {
    Ordering<int, double> x;      // base template
    Ordering<double*, double*> y; // Ordering<T*,U*>
    Ordering<int*, double*> z;    // Ordering<int*, double*>

    static_assert(x.v == 0);
    static_assert(y.v == 1);
    static_assert(z.v == 2);
}
```

这段代码展示了 C++ 中模板特化的用法，包括类型模板（struct）和变量模板（constexpr inline）的特化规则。代码通过示例和注释说明了模板特化的合法性、限制以及选择规则。以下是对代码的详细解释：

------

**代码结构**

1. **类型模板特化**：
   - 定义主模板 Type 和 NonType，并展示部分特化和完全特化的规则。
   - 定义 Ordering，展示多级特化选择。
2. **变量模板特化**：
   - 定义 ID，展示变量模板的部分特化。
3. **验证**：
   - 在 main 中使用 static_assert 检查特化选择。

------

**代码逐步解释**

**1. 主模板 Type 和部分特化**

cpp

```cpp
template <typename T, typename U>
struct Type {};

// 合法的部分特化
template <typename X>
struct Type<int, X> {};
```

- **Type<T, U>**：
  - 主模板，接受两个类型参数 T 和 U。
- **Type<int, X>**：
  - 部分特化，将 T 固定为 int，U 仍为模板参数 X。
  - 合法，因为特化了主模板的一个参数。

cpp

```cpp
/* 不合法：未特化任何参数
template <typename X, typename Y>
struct Type<X, Y> {};
*/
```

- **问题**：
  - 这只是主模板的重定义，未特化任何参数。
  - C++ 要求特化必须约束或固定至少一个模板参数。

cpp

```cpp
/* 不合法：特化中不允许默认参数
template <typename X = int>
struct Type<X, X> {};
*/
```

- **问题**：

  - 模板特化不允许使用默认参数。

  - 默认参数只能用于主模板，例如：

    cpp

    ```cpp
    template <typename X = int, typename Y>
    struct Type {};
    ```

------

**2. 非类型模板参数 NonType**

cpp

```cpp
template <typename Type, Type Value>
struct NonType {};

// 合法的完全特化
template <>
struct NonType<int, 4> {};
```

- **NonType<Type, Value>**：
  - 主模板，接受类型参数 Type 和该类型的非类型参数 Value。
- **NonType<int, 4>**：
  - 完全特化，固定 Type = int 和 Value = 4。
  - 合法，因为明确指定了所有参数。

cpp

```cpp
/* 不合法：依赖参数不能自由特化
template <typename X>
struct NonType<X, 4> {};
*/
```

- **问题**：
  - 非类型参数 Value 依赖于类型参数 X，不能单独特化 Value 而保留 X 为模板参数。
  - C++ 要求非类型参数的类型在特化时完全确定。

------

**3. 特化选择规则 Ordering**

cpp

```cpp
template <typename T, typename U>
struct Ordering { static constexpr int v = 0; };

template <typename T, typename U>
struct Ordering<T*, U*> { static constexpr int v = 1; };

template <>
struct Ordering<int*, double*> { static constexpr int v = 2; };
```

- **Ordering<T, U>**：
  - 主模板，v = 0。
- **Ordering<T\*, U\*>**：
  - 部分特化，要求 T 和 U 是指针类型，v = 1。
- **Ordering<int\*, double\*>**：
  - 完全特化，固定为 int* 和 double*，v = 2。
- **选择规则**：
  - C++ 编译器选择“最特化”的模板：
    - 主模板：最通用。
    - T*, U*：更具体（要求指针）。
    - int*, double*：最具体（固定类型）。

------

**4. 变量模板特化 ID**

cpp

```cpp
template <typename X, typename Y>
constexpr inline std::size_t ID = 0;

template <typename Y>
constexpr inline std::size_t ID<Y, int> = 10;
```

- **ID<X, Y>**：
  - 主变量模板，值为 0。
- **ID<Y, int>**：
  - 部分特化，固定 Y 为 int，值为 10。
- **用法**：
  - ID<double, int> 匹配特化，结果为 10。
  - ID<double, double> 匹配主模板，结果为 0。

------

**5. main 函数验证**

cpp

```cpp
int main() {
    Ordering<int, double> x;      // 主模板
    Ordering<double*, double*> y; // Ordering<T*, U*>
    Ordering<int*, double*> z;    // Ordering<int*, double*>

    static_assert(x.v == 0);
    static_assert(y.v == 1);
    static_assert(z.v == 2);
}
```

- **x**：
  - Ordering<int, double> 匹配主模板，v = 0。
- **y**：
  - Ordering<double*, double*> 匹配 T*, U*（T = double, U = double），v = 1。
- **z**：
  - Ordering<int*, double*> 匹配完全特化，v = 2。
- **static_assert**：
  - 编译期验证，确保特化选择正确。

------

**关键技术点**

1. **模板特化规则**：
   - 部分特化必须约束至少一个参数。
   - 完全特化固定所有参数。
2. **非类型参数限制**：
   - 依赖类型参数的非类型参数不能单独特化。
3. **特化优先级**：
   - 编译器选择最具体的匹配模板。
4. **变量模板**：
   - C++14 引入，特化规则与类型模板类似。

------

**可能的改进或注意事项**

1. **注释准确性**：

   - 代码注释清晰，但可添加特化选择的例子。

2. **扩展验证**：

   - 可测试 ID 的值，例如：

     cpp

     ```cpp
     static_assert(ID<double, int> == 10);
     static_assert(ID<double, double> == 0);
     ```

3. **错误示例**：

   - 注释中的不合法代码可移到注释外，展示编译错误。

------

**总结**

- **类型特化**：Type 和 NonType 展示合法和不合法的特化。
- **选择规则**：Ordering 演示多级特化的优先级。
- **变量特化**：ID 展示变量模板的用法。
- **验证**：通过 static_assert 确认行为。

如果你有具体问题（例如特化匹配的细节），欢迎提问！