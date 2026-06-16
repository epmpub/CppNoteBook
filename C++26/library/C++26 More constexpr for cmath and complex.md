C++26 “More constexpr for <cmath> and <complex>”

提案 P1383R2）是 C++26 中 constexpr 支持的重要扩展，显著提升了编译期数学计算的能力。 

sandordargo.com

背景

- C++23（P0533）首次让部分 <cmath> 函数成为 constexpr，但仅限于“复杂度不高于基本算术运算”的简单函数（如 abs、floor、ceil、trunc、round、fmin、fmax、fma、hypot 等）。
- C++26（P1383R2）移除了这个限制，允许更多复杂数学函数在常量表达式中使用，包括三角函数、指数、对数等。

主要变化在 <cmath> 中，大量常用函数现在可以标记为 constexpr：

- 三角函数：sin、cos、tan、asin、acos、atan、atan2
- 双曲函数：sinh、cosh、tanh、asinh、acosh、atanh
- 指数与对数：exp、exp2、log、log10、log2、log1p、expm1
- 幂与根：pow、sqrt、cbrt
- 其他：erf、erfc、lgamma、tgamma、remainder、remquo、nextafter、nexttoward、fdim 等（部分已有实现支持）

在 <complex> 中，相应的复数数学函数也扩展为 constexpr：

- sin、cos、tan、asin、acos、atan
- sinh、cosh、tanh、asinh、acosh、atanh
- exp、log、log10、pow、sqrt、abs 等

使用示例

cpp

```cpp
#include <cmath>
#include <complex>
#include <iostream>

constexpr double compile_time_sin(double x) {
    return std::sin(x);
}

constexpr std::complex<double> compile_time_complex_exp(double x) {
    return std::exp(std::complex<double>{0.0, x});  // e^(ix)
}

int main() {
    constexpr double s = compile_time_sin(3.1415926535 / 2.0);
    static_assert(s > 0.999 && s < 1.001);   // 近似 π/2 的 sin 值

    constexpr auto c = compile_time_complex_exp(3.1415926535);  // e^(iπ) ≈ -1
    static_assert(std::abs(c + 1.0) < 1e-9);

    std::cout << "Compile-time sin(π/2) ≈ " << s << '\n';
}
```

设计要点与注意事项

- 浮点数不确定性：浮点计算在不同平台、编译选项（-ffast-math 等）和优化级别下可能产生微小差异。标准接受这种 QoI（Quality of Implementation）差异，不强制要求“完美舍入到最后一位”。

- 编译器实现：依赖编译器内置函数（builtins）或库实现。GCC/Clang/MSVC 已在逐步支持，部分函数可能需要最新版本。

- 特性测试宏：

  cpp

  ```cpp
  #ifdef __cpp_lib_constexpr_cmath
      // 202306L（或更新值）
  #endif
  
  #ifdef __cpp_lib_constexpr_complex
      // 202306L（C++26 版本）
  #endif
  ```

- 适用场景：

  - 编译期计算查找表（lookup tables）
  - 常量表达式中的物理模拟、信号处理、图形学计算
  - constexpr 容器 + 数学计算的结合（C++26 其他特性）
  - 元编程中生成数学常数

这个提案让 C++ 的编译期计算能力又向前迈了一大步，尤其对数值计算和模板元编程开发者非常友好。 

cppreference.com

更多细节可参考：

- 提案 [P1383R2](https://wg21.link/P1383)
- cppreference C++26 特性列表（已标记该特性）