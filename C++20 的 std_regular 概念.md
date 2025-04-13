# C++20 的 std::regular 概念

`std::regular` 是 C++20 引入的一个概念(concept)，用于描述一种具有常规值语义(regular value semantics)的类型。它是标准库中定义的最严格的值类别之一。

## std::regular 的定义

在 `<concepts>` 头文件中，`std::regular` 定义为：

```cpp
template <class T>
concept regular = std::semiregular<T> && std::equality_comparable<T>;
```

这意味着一个 `regular` 类型必须满足：
1. `semiregular` 的要求（即可默认构造和可拷贝）
2. `equality_comparable` 的要求（即可进行相等性比较）

## 具体要求

一个满足 `std::regular` 的类型 T 必须满足以下条件：

1. **可默认构造**：`T{}` 是合法表达式
2. **可拷贝构造**：可以从另一个 T 对象构造新对象
3. **可拷贝赋值**：可以将一个 T 对象赋值给另一个
4. **可析构**：对象可以被正确销毁
5. **可相等比较**：可以使用 `==` 和 `!=` 运算符比较两个 T 对象
6. **遵循常规语义**：
   - 拷贝后的对象与原对象相等
   - 赋值是破坏性操作（赋值后原值被覆盖）
   - 相等性比较遵循等价关系（自反性、对称性、传递性）

## 为什么重要

`std::regular` 类型非常重要，因为它们：

- 行为类似于内置类型（如 int、double）
- 可以安全地用于标准容器和算法
- 可以按值传递和返回而不会引起问题
- 可以存储在容器中并支持查找、排序等操作

## 示例

```cpp
#include <concepts>
#include <string>

struct Point {
    int x, y;
    
    // 默认构造函数
    Point() : x(0), y(0) {}
    
    // 比较运算符
    bool operator==(const Point&) const = default;
};

static_assert(std::regular<Point>);  // Point 是 regular 类型
static_assert(std::regular<std::string>);  // std::string 也是 regular 类型
static_assert(!std::regular<int&>);  // 引用不是 regular 类型
```

## 与其他概念的关系

- `std::semiregular`：可默认构造和可拷贝，但不要求可比较
- `std::equality_comparable`：可比较相等性
- `std::totally_ordered`：除了相等性，还支持 `<`, `>`, `<=`, `>=`
- `std::regular_invocable`：可调用的概念，与 regular 类型相关

`std::regular` 是许多标准算法和容器对元素类型的基本要求之一。