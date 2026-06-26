C++26 “Adding the new 2022 SI prefixes on ratios”

（提案 P2734R0）在 <ratio> 头文件中新增了四个类型别名，对应 2022 年国际计量大会（CGPM）正式采纳的四个新 SI 前缀。新增的四个 std::ratio 别名（C++26）

| 类型别名    | SI 前缀 | 乘数因子 | 符号 | 说明         |
| ----------- | ------- | -------- | ---- | ------------ |
| std::quecto | quecto  | 10⁻³⁰    | q    | 极小（最小） |
| std::ronto  | ronto   | 10⁻²⁷    | r    | 极小         |
| std::ronna  | ronna   | 10²⁷     | R    | 极大         |
| std::quetta | quetta  | 10³⁰     | Q    | 极大（最大） |

这些都是 std::ratio 的特化别名，与已有的 std::kilo、std::milli、std::pico 等完全一致。代码示例

cpp

```cpp
#include <ratio>
#include <iostream>

int main() {
    using namespace std::literals::chrono_literals; // 示例用途

    // 使用新前缀
    using quettagram  = std::quetta<std::ratio<1>>;   // 10³⁰ gram
    using rontosecond = std::ronto<std::ratio<1>>;    // 10⁻²⁷ second

    std::cout << "1 quettagram  = " << quettagram::num << " / " << quettagram::den << " gram\n";
    std::cout << "1 rontosecond = " << rontosecond::num << " / " << rontosecond::den << " second\n";

    // 与现有前缀混合使用（编译期计算）
    using mass = std::ratio_multiply<std::quetta<std::ratio<5>>, std::kilo>; // 5 × 10³³ gram
}
```

主要设计要点

- 纯添加：只新增四个类型别名，不改变任何现有接口。

- 编译期：所有运算仍在编译期完成，std::ratio 的所有元函数（ratio_add、ratio_multiply、ratio_divide 等）均可正常使用。

- 特性测试宏：

  cpp

  ```cpp
  #ifdef __cpp_lib_ratio
      // C++26 中该宏的值会被更新
  #endif
  ```

- 命名规则：符合 SI 前缀的命名习惯（大于 kilo 的以 -a 结尾，小于 milli 的以 -o 结尾）。

实际应用场景

- 高能物理、宇宙学、天文学（极大质量、距离、能量）
- 量子计算、粒子物理、纳米/飞秒级以下的精密测量（极小时间、质量）
- 科学计算库、单位库（std::chrono、std::units 等自定义单位系统）
- 需要精确表达极端数量级的编译期常量

这个特性虽然很小，但体现了 C++ 标准库与国际科学标准保持同步的理念，让开发者可以直接在编译期使用最新的 SI 单位前缀，而无需自己手动定义 std::ratio<1, 1000000000000000000000000000000LL> 这样容易出错的代码。更多细节可参考：

- 提案 [P2734R0](https://wg21.link/P2734)
- [cppreference: std::ratio](https://en.cppreference.com/cpp/numeric/ratio/ratio)（C++26 已更新）