# 平凡可复制类型（Trivially Copyable）

这段代码和注释展示了 C++ 中**平凡可复制类型（Trivially Copyable）**的定义和特性，并通过三个例子（X、Y、Z）来说明哪些类型符合或不符合这一标准。以下是对代码的详细解释：

```c++
#include <numeric>
#include <array>
#include <string>
#include <span>

// Trivially copyable class-types:
// - one eligible trivial copy/move-constr. or copy/move-assign.
// - no eligible non-trivial copy/move-constr. and copy/move-assign.
// - non-deleted trivial destructor

// Trivial special member function (constr., assign., destr.):
// - implicit or defaulted
// - no virtual members or bases
// - recursively trivial:
//   - all bases and non-static members have corresponding
//     trivial constr./assign./destr.

// Trivially copyable example:
// ✔ implicit copy&move constructors && copy&move assignments
// ✔ implicit destructor
// ✔ no virtual members or bases
// ✔ all members are trivially copyable
// ✔ all bases are trivially copyable
struct X {
    X() { std::iota(data.begin(), data.end(), 1); }
    std::span<int> get_buff() { return data; }
private:
    std::array<int, 42> data;
};

// Trivially copyable despite being move-only:
// ✔ defaulted move-constructor && move-assignment
// ✔ implicit destructor
// ✔ no virtual members or bases
// ✔ all members are trivially copyable
// ✔ all bases are trivially copyable
struct Y {
    Y(const Y&) = delete;
    Y& operator=(const Y&) = delete;
    Y(Y&&) = default;
    Y& operator=(Y&&) = default;
private:
    int x;
    float y;
};

// Not trivially copyable
// ❌ std::string is not trivially copyable
struct Z {
    std::string text;
};

int main() {
    static_assert(std::is_trivially_copyable_v<X>);
    static_assert(std::is_trivially_copyable_v<Y>);
    static_assert(!std::is_copy_constructible_v<Y> && 
        !std::is_copy_assignable_v<Y>);
    static_assert(!std::is_trivially_copyable_v<Z>);
}
```



------

什么是“平凡可复制类型”？

在 C++ 中，一个类型是**平凡可复制的（trivially copyable）**，意味着它可以安全地通过低级操作（如 memcpy）进行复制或移动，而不需要执行复杂的用户定义逻辑。标准对此有明确的定义：

定义条件

1. **至少有一个平凡的复制/移动构造函数或赋值操作**：
   - 类型必须有一个可用的平凡复制构造函数、移动构造函数、复制赋值运算符或移动赋值运算符。
2. **没有非平凡的复制/移动构造函数和赋值操作**：
   - 如果定义了这些特殊成员函数，它们必须是平凡的，不能有用户提供的复杂逻辑。
3. **平凡的析构函数**：
   - 析构函数必须是平凡的（默认或隐式的，且未被删除）。
4. **递归平凡性**：
   - 所有基类和非静态数据成员也必须是平凡可复制的。

平凡特殊成员函数的条件

一个特殊成员函数（构造函数、赋值运算符、析构函数）是**平凡的（trivial）**，需要满足：

- 是隐式定义的（implicit）或默认定义的（= default）。
- 没有虚函数或虚基类。
- 所有基类和非静态成员的对应特殊成员函数也必须是平凡的。

平凡可复制类型的意义在于，它们可以被编译器优化为简单的内存拷贝操作，且在某些场景（如 std::memcpy 或跨进程传递）中是安全的。

------

示例分析

1. struct X

cpp

```cpp
struct X {
    X() { std::iota(data.begin(), data.end(), 1); }
    std::span<int> get_buff() { return data; }
private:
    std::array<int, 42> data;
};
```

- **分析**：
  - **复制/移动构造函数和赋值运算符**：没有显式定义，因此是隐式的，默认提供平凡的复制和移动操作。
  - **析构函数**：隐式定义，是平凡的。
  - **虚成员或基类**：没有。
  - **成员**：std::array<int, 42> 是平凡可复制的（固定大小数组，其元素 int 是平凡类型）。
  - **构造函数**：有一个用户定义的默认构造函数，但这不影响平凡可复制性，因为平凡可复制性只关心复制/移动和析构行为。
- **结论**：X 是平凡可复制的。
  - 它的复制和移动操作是平凡的（逐字节复制 data），尽管构造时有自定义逻辑。
- struct Y

cpp

```cpp
struct Y {
    Y(const Y&) = delete;
    Y& operator=(const Y&) = delete;
    Y(Y&&) = default;
    Y& operator=(Y&&) = default;
private:
    int x;
    float y;
};
```

- **分析**：
  - **复制构造函数和赋值运算符**：被显式删除（= delete），因此不可用。
  - **移动构造函数和赋值运算符**：显式默认（= default），是平凡的。
  - **析构函数**：隐式定义，是平凡的。
  - **虚成员或基类**：没有。
  - **成员**：int 和 float 都是平凡可复制的。
  - **关键点**：虽然 Y 是“仅移动（move-only）”类型，但它仍然满足平凡可复制的定义，因为它有一个平凡的移动构造函数和移动赋值运算符。
- **结论**：Y 是平凡可复制的。
  - 它不能被复制，但可以被平凡地移动（本质上是内存拷贝）。
- struct Z

cpp

```cpp
struct Z {
    std::string text;
};
```

- **分析**：
  - **复制/移动构造函数和赋值运算符**：隐式定义，但依赖于 std::string 的实现。
  - **析构函数**：隐式定义，但依赖于 std::string 的析构函数。
  - **虚成员或基类**：没有。
  - **成员**：std::string 不是平凡可复制的，因为：
    - 它有非平凡的复制构造函数（需要分配内存）。
    - 它有非平凡的析构函数（需要释放内存）。
  - **递归平凡性**：由于 std::string 不满足平凡性，Z 也不满足。
- **结论**：Z 不是平凡可复制的。
  - 它的复制和析构操作涉及复杂的动态内存管理。

------

关键点总结

1. **平凡可复制的核心**：
   - 关注复制/移动和析构的平凡性，而不是构造逻辑。
   - 要求所有相关操作可以简化为内存拷贝。
2. **X 的情况**：
   - 用户定义的构造函数不影响平凡可复制性，只要复制/移动和析构是平凡的。
3. **Y 的情况**：
   - 即使是仅移动类型，只要移动操作是平凡的，仍可视为平凡可复制。
4. **Z 的情况**：
   - 如果成员类型（如 std::string）是非平凡的，整个类型就不再平凡可复制。

------

使用场景

- **平凡可复制类型**适合：
  - 通过 memcpy 或 memmove 进行低级复制。
  - 在性能敏感的代码中避免不必要的开销。
  - 在需要序列化或跨边界传递数据时（例如 C 风格接口）。
- **非平凡类型**（如 Z）：
  - 需要手动管理资源（如动态内存），无法简单地按字节复制。

希望这个解释清楚地阐明了平凡可复制类型的规则和这些例子的特性！