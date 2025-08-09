# [[no_unique_address]] 

在 C++ 中，[[no_unique_address]] 是一个 C++20 引入的属性（attribute），用于优化类或结构体的内存布局。它允许编译器对某个非静态数据成员不分配唯一的内存地址，从而在某些情况下减少对象的大小。这种属性特别适用于空基类优化（Empty Base Optimization, EBO）或空成员优化的场景。

以下是对 [[no_unique_address]] 的详细解释：

```c++
#include <utility>

struct Empty {};

struct Bigger {
    int value;
    // Even though empty, has to take up at least one byte.
    Empty empty;
};

struct SameAsInt {
    int value;
    // Allowed to overlap.
    [[no_unique_address]] Empty empty;
};

int main() {
    static_assert(sizeof(Empty) >= 1);
    static_assert(sizeof(Bigger) >= sizeof(int) + 1);
    static_assert(sizeof(SameAsInt) == sizeof(int));
}
```



------

定义

cpp

```cpp
[[no_unique_address]] 类型 成员名;
```

- **[[no_unique_address]]** 是一个属性，告诉编译器该成员不需要独立的内存地址。
- 适用于类的非静态数据成员。
- 编译器可以选择将该成员与其它成员重叠存储，前提是这样做是合法的。

------

行为

- **默认规则**：
  - 在 C++ 中，每个非静态数据成员通常都有唯一的内存地址，即使它是一个空类或结构体（大小为 0）。
  - 这会导致额外的填充（padding）或空间浪费。
- **[[no_unique_address]]**：
  - 允许编译器在布局时忽略该成员的地址唯一性。
  - 如果成员是空类型（大小为 0，例如空类），它可能不占用额外空间，而是与其它成员共享地址。
- **限制**：
  - 如果成员有非平凡的析构函数、构造函数或需要实际存储数据，编译器仍需为其分配空间。
  - 效果依赖于编译器实现，不保证一定优化。

------

前提条件

- **适用对象**：
  - 通常用于空类或不占用空间的类型（如空状态对象）。
- **要求**：
  - 成员必须是非静态数据成员。
  - 不影响程序的语义（即，不能依赖成员的地址唯一性）。

------

示例代码

示例 1：基本用法

cpp

```cpp
#include <iostream>

struct Empty {
    // 空类
};

struct Data {
    int value = 42;
    [[no_unique_address]] Empty e; // 不需要独立地址
};

int main() {
    std::cout << "Size of Empty: " << sizeof(Empty) << "\n"; // 通常为 1
    std::cout << "Size of Data: " << sizeof(Data) << "\n";   // 可能为 4
    return 0;
}
```

输出（依赖编译器）

```text
Size of Empty: 1
Size of Data: 4
```

- **解释**：
  - Empty 是空类，默认大小为 1（避免零大小对象的地址问题）。
  - Data 中，e 被标记为 [[no_unique_address]]，编译器可能将其与 value 重叠存储。
  - 结果：sizeof(Data) 可能等于 sizeof(int)（4 或 8，视平台而定），而不是 sizeof(int) + sizeof(Empty)。

示例 2：与非空类比较

cpp

```cpp
#include <iostream>

struct Empty {};

struct NonEmpty {
    int x = 0;
};

struct Test {
    int value = 42;
    [[no_unique_address]] Empty e1;
    NonEmpty n;
};

int main() {
    std::cout << "Size of Test: " << sizeof(Test) << "\n";
    return 0;
}
```

输出（示例）

```text
Size of Test: 8
```

- **解释**：
  - e1 是空类，[[no_unique_address]] 使其可能不占空间。
  - n 是 NonEmpty，占用 4 字节。
  - value 占用 4 字节。
  - 总大小可能为 8（4 + 4），e1 被优化掉。

示例 3：函数对象优化

cpp

```cpp
#include <iostream>
#include <functional>

struct Functor {
    int operator()(int x) const { return x * 2; }
};

struct Holder {
    int data = 10;
    [[no_unique_address]] Functor f; // 无状态函数对象
};

int main() {
    Holder h;
    std::cout << "Size of Holder: " << sizeof(h) << "\n";
    std::cout << "Result: " << h.f(h.data) << "\n";
    return 0;
}
```

输出（示例）

```text
Size of Holder: 4
Result: 20
```

- **解释**：
  - Functor 是无状态的，[[no_unique_address]] 允许其不占用空间。
  - Holder 大小可能仅为 int（4 字节）。

------

与空基类优化 (EBO) 的关系

- **EBO**：
  - C++ 允许空基类不占用额外空间（即使基类默认大小为 1）。
  - 不需要显式属性。
- **[[no_unique_address]]**：
  - 将类似优化扩展到非静态数据成员。
  - 更通用，不限于基类。

EBO 示例

cpp

```cpp
struct Base {};
struct Derived : Base {
    int x;
};
```

- sizeof(Derived) == sizeof(int)，无需 [[no_unique_address]]。

------

时间复杂度与空间优化

- **时间复杂度**：
  - 无影响，仅影响编译期布局。
- **空间复杂度**：
  - 可能减少对象大小，具体节省依赖成员类型和编译器。

------

使用场景

1. **优化内存布局**：

   - 在类中包含无状态对象（如函数对象、标记类型）。

   cpp

   ```cpp
   struct Config {
       int value;
       [[no_unique_address]] std::less<int> cmp;
   };
   ```

2. **嵌入式系统**：

   - 减少不必要内存占用。

3. **模板编程**：

   - 在泛型代码中避免空类型增加大小。

------

注意事项

1. **C++20 要求**：
   - 需要 -std=c++20 和支持 C++20 的编译器。
2. **编译器依赖**：
   - 优化是可选的，不保证生效。
   - 检查 sizeof 验证效果。
3. **地址重叠**：
   - 不能依赖成员的独立地址（如 &e != &value 可能不成立）。
4. **非空类型**：
   - 如果成员非空，[[no_unique_address]] 无效果。

------

与其他属性的对比

| 属性                  | 作用             | 适用场景   |
| --------------------- | ---------------- | ---------- |
| [[no_unique_address]] | 允许成员地址重叠 | 空成员优化 |
| [[maybe_unused]]      | 抑制未使用警告   | 未使用变量 |
| [[nodiscard]]         | 要求使用返回值   | 函数返回值 |

------

总结

[[no_unique_address]] 是 C++20 的属性，用于优化内存布局，允许空成员不占用独立空间。它扩展了空基类优化的思想，适用于无状态成员或空类型，能有效减少对象大小。效果依赖编译器实现，需验证实际优化。如果你有具体问题或想探讨用法，请告诉我！