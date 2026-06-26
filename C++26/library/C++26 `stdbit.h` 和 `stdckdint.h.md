**C++26 `<stdbit.h>` 和 `<stdckdint.h>`** 是为了提升与 **C23** 的兼容性而引入的两个**兼容性头文件**（Compatibility Headers），通过提案 **P3370R1** 落地。

它们直接对应 C23 的同名头文件，**不提供 `<cstdbit>` / `<cstdckdint>`** 版本（即没有 `std::` 命名空间下的对应物）。这是为了让 C 和 C++ 代码在需要跨语言兼容时直接使用相同的头文件。

### 1. `<stdbit.h>` —— 位操作函数（Bit Manipulation）

这个头文件提供了一系列**类型泛型**（type-generic）的位操作函数，主要用于计算**前导/尾随零/一**的数量、首个零/一的位置、零/一的总数等。

**主要特性**：
- 支持 `unsigned char`、`unsigned short`、`unsigned int`、`unsigned long`、`unsigned long long` 的具体函数。
- 提供 **泛型模板** 重载（`template<class T> auto stdc_xxx(T value)`），自动选择合适实现。
- 所有函数 `constexpr` 友好且性能极高（通常编译为单个 CPU 指令，如 `lzcnt`、`tzcnt`、`popcnt`）。

**关键函数**（部分）：

- `stdc_leading_zeros` / `stdc_leading_ones`
- `stdc_trailing_zeros` / `stdc_trailing_ones`
- `stdc_first_leading_zero` / `stdc_first_leading_one`
- `stdc_first_trailing_zero` / `stdc_first_trailing_one`
- `stdc_count_zeros` / `stdc_count_ones`
- 字节序宏：`__STDC_ENDIAN_BIG__`、`__STDC_ENDIAN_LITTLE__`、`__STDC_ENDIAN_NATIVE__`

**示例**：

```cpp
#include <stdbit.h>
#include <cstdint>
#include <cstdio>

int main() {
    uint32_t x = 0b00001000'00000000'00000000'00000000;  // 只有第 27 位是 1（从 0 开始）

    printf("leading zeros: %u\n", stdc_leading_zeros_ui(x));   // 3（或用泛型）
    printf("trailing zeros: %u\n", stdc_trailing_zeros(x));    // 27
    printf("popcount: %u\n", stdc_count_ones(x));              // 1
}
```

**与 `<bit>` 的关系**：
- C++20 的 `<bit>`（`std::countl_zero`、`std::countr_zero`、`std::popcount` 等）功能高度重叠。
- `<stdbit.h>` 主要服务**C 兼容性**和**遗留代码迁移**，名字风格为 `stdc_` 前缀。

**Feature Test Macro**：
```cpp
#if __cpp_lib_stdbit_h >= 202603L  // 或类似值
```

### 2. `<stdckdint.h>` —— 检查整数运算（Checked Integer Arithmetic）

提供**带溢出检测**的加、减、乘操作，极大提升整数运算的安全性（尤其在安全关键代码中）。

**核心函数**（均为函数模板）：

```cpp
template<class type1, class type2, class type3>
bool ckd_add(type1* result, type2 a, type3 b);

template<class type1, class type2, class type3>
bool ckd_sub(type1* result, type2 a, type3 b);

template<class type1, class type2, class type3>
bool ckd_mul(type1* result, type2 a, type3 b);
```

- **行为**：以**无限精度**计算 `a op b`，然后转换到 `type1`。
- **返回值**：`false` 表示没有溢出（结果精确），`true` 表示溢出（`*result` 得到 wrap-around 后的值）。
- 支持任意整数类型（signed/unsigned）的组合。

**示例**（来自 cppreference）：

```cpp
#include <stdckdint.h>
#include <cstdint>
#include <limits>
#include <cstdio>

int main() {
    std::uint8_t x = 14;
    std::uint16_t y = 28, result{};
    bool overflow;

    overflow = ckd_add(&result, x, y);           // 42 → OK
    printf("%d + %d = %d (%s)\n", x, y, result, overflow ? "overflow" : "OK");

    y = std::numeric_limits<std::uint16_t>::max();
    overflow = ckd_add(&result, x, y);           // 溢出
    printf("%d + %d = %d (%s)\n", x, y, result, overflow ? "overflow" : "OK");
}
```

**Feature Test Macro**：
```cpp
#if __cpp_lib_stdckdint_h >= 202603L
```

### 设计目的与影响

- **C/C++ 互操作**：让跨语言项目（尤其是嵌入式、系统编程、内核）更容易共享代码。
- **安全性**：`<stdckdint.h>` 显著降低了整数溢出漏洞的风险。
- **性能**：这些操作通常被编译器优化为原生指令（GCC/Clang/MSVC 已支持类似内置函数多年）。
- **实现状态**：GCC、Clang、MSVC 等主流编译器已在快速跟进 C++26 支持。

更多细节可参考：
- cppreference: [`<stdbit.h>`](https://en.cppreference.com/cpp/header/stdbit.h)、[`<stdckdint.h>`](https://en.cppreference.com/cpp/header/stdckdint.h)
- 提案 P3370R1

这两个头文件是 C++26 “更好地拥抱 C23” 策略的一部分，让 C++ 在底层编程场景中更具竞争力。