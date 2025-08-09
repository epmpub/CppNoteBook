# Unscoped Enum

```C++
#include <cstdint>
#include <numeric>

// Enumerators are assigned values starting from 0,
// unless overriden, the Enumerators are assgned consecutive values
enum Vegetables { Potato, Tomato, Carrot = 42, Pea, Onion };
// Potato == 0, Tomato == 1, Carrot = 42, Pea == 43, Onion == 44

enum Specific {
    A = 1,
    B = A + 42,
    C = 1, // values may repeat
    D = A + B + C,
    E = std::lcm(13,7) // any constant expression
};
// A == 1, B == 43, C == 1, D == 45, E == 91

int v = A; // Enumerators are implicitly convertible 
           // to the backing type
Specific w = static_cast<Specific>(1);
// To convert from the underlying type we need a static cast
// Note, multiple enumerators with the same value do not matter
// because they are not distinguishable (i.e. A == C)


// Enumerations can be anonymous 
// and explicitly specify the backing type
enum : unsigned { GlobalConstant = 42 };
// GlobalConstant == 42
// std::underlying_type_t<decltype(GlobalConstant)> == unsigned


// A type that can represent all enumeration values is picked
enum IntLimits {
    Min64Bit = INT64_MIN,
    Max64Bit = INT64_MAX
};
// std::underlying_type_t<IntLimits> == 64bit type

// If no such type exists, the enumeration is ill-formed
enum Impossible {
    X = INT64_MIN,
    Y = UINT64_MAX
};
// Note GCC actually silently picks __int128
```



这段代码展示了 C++ 中枚举（enum）的特性，包括枚举值的赋值规则、底层类型、类型转换以及一些特殊情况。以下是对代码的逐步解释。

------

代码概览

- 展示了普通枚举（Vegetables 和 Specific）的赋值规则。
- 介绍了匿名枚举和显式底层类型的用法。
- 探讨了枚举的底层类型选择规则，包括超出标准类型范围的情况。

------

关键组件

1. **头文件**

cpp

```cpp
#include <cstdint>
#include <numeric>
```

- <cstdint>：提供固定大小的整数类型（如 int64_t、uint64_t）。
- <numeric>：提供 std::lcm（最小公倍数），用于计算枚举值。
- **普通枚举：Vegetables**

cpp

```cpp
enum Vegetables { Potato, Tomato, Carrot = 42, Pea, Onion };
// Potato == 0, Tomato == 1, Carrot = 42, Pea == 43, Onion == 44
```

- **规则**：
  - 枚举值从 0 开始递增，除非显式指定。
  - 显式赋值后，后续值继续递增。
- **赋值**：
  - Potato = 0（默认起始值）。
  - Tomato = 1（递增）。
  - Carrot = 42（显式指定）。
  - Pea = 43（从 Carrot 递增）。
  - Onion = 44（继续递增）。
- **底层类型**：
  - 未显式指定，编译器选择能表示所有值的最小整数类型（这里可能是 int）。
- **复杂枚举：Specific**

cpp

```cpp
enum Specific {
    A = 1,
    B = A + 42,
    C = 1, // values may repeat
    D = A + B + C,
    E = std::lcm(13,7) // any constant expression
};
// A == 1, B == 43, C == 1, D == 45, E == 91
```

- **赋值**：
  - A = 1（显式指定）。
  - B = A + 42 = 1 + 42 = 43（使用先前枚举值计算）。
  - C = 1（显式指定，允许重复）。
  - D = A + B + C = 1 + 43 + 1 = 45（组合计算）。
  - E = std::lcm(13,7) = 91（编译期常量表达式，13 和 7 的最小公倍数）。
- **特性**：
  - 枚举值可以重复（A == C），但它们不可区分（仅值相同）。
  - 支持任意编译期常量表达式（如 std::lcm）。
- **类型转换**

cpp

```cpp
int v = A; // Enumerators are implicitly convertible to the backing type
Specific w = static_cast<Specific>(1); // To convert from the underlying type we need a static cast
```

- **从枚举到整数**：
  - 枚举值（如 A）可隐式转换为底层类型（这里是 int）。
  - v = A = 1。
- **从整数到枚举**：
  - 需要显式 static_cast，因为枚举是强类型。
  - w = static_cast<Specific>(1)，可能是 A 或 C（值相同，无法区分）。
- **注意**：
  - 重复值不影响转换，但具体枚举值未定义（A 或 C 均可）。
- **匿名枚举与显式底层类型**

cpp

```cpp
enum : unsigned { GlobalConstant = 42 };
// GlobalConstant == 42
// std::underlying_type_t<decltype(GlobalConstant)> == unsigned
```

- **匿名枚举**：
  - 没有名称，仅定义常量。
  - GlobalConstant 是全局常量。
- **显式底层类型**：
  - 使用 : unsigned 指定底层类型为 unsigned int。
- **验证**：
  - std::underlying_type_t 确认底层类型是 unsigned。
- **底层类型自动选择：IntLimits**

cpp

```cpp
enum IntLimits {
    Min64Bit = INT64_MIN,
    Max64Bit = INT64_MAX
};
// std::underlying_type_t<IntLimits> == 64bit type
```

- **值**：
  - INT64_MIN = -2^63（-9,223,372,036,854,775,808）。
  - INT64_MAX = 2^63 - 1（9,223,372,036,854,775,807）。
- **底层类型**：
  - 编译器选择能表示所有值的类型。
  - 这里需要 64 位有符号整数（int64_t），因为值超出 32 位范围。
- **超出范围的枚举：Impossible**

cpp

```cpp
enum Impossible {
    X = INT64_MIN,
    Y = UINT64_MAX
};
// Note GCC actually silently picks __int128
```

- **值**：
  - INT64_MIN = -2^63。
  - UINT64_MAX = 2^64 - 1（18,446,744,073,709,551,615）。
- **问题**：
  - 标准 C++ 中没有单一整数类型能同时表示 INT64_MIN（负值）和 UINT64_MAX（最大无符号值）。
  - 需要至少 65 位有符号整数，但标准类型最大为 64 位。
- **标准行为**：
  - 这种枚举是“格式不正确”（ill-formed），应导致编译错误。
- **GCC 行为**：
  - GCC 扩展使用 __int128（128 位整数）默默支持，避免错误。
  - 这不是标准行为，依赖实现。

------

为什么这样工作？

1. **枚举值规则**：
   - 从 0 开始递增，可显式覆盖，支持常量表达式。
2. **底层类型**：
   - 默认由编译器选择，通常是 int，除非显式指定或值超出范围。
   - 必须能表示所有枚举值，否则格式不正确。
3. **类型转换**：
   - 枚举到整数是隐式的，反向需要显式转换。
4. **实现依赖**：
   - 如 Impossible，超出标准范围时依赖编译器扩展。

------

输出

- 没有运行时输出，仅展示编译期特性。

------

使用场景

- **常量定义**：如 Vegetables 和 Specific，定义一组相关值。
- **全局常量**：匿名枚举（如 GlobalConstant）替代宏。
- **大范围值**：如 IntLimits，表示特定类型边界。

------

总结

- Vegetables 和 Specific 展示了枚举赋值和计算。
- 类型转换展示了枚举的强类型特性。
- 匿名枚举和显式底层类型提供了灵活性。
- Impossible 揭示了枚举范围的限制和实现依赖性。
- 这段代码是理解 C++ 枚举规则和底层类型选择的良好示例。