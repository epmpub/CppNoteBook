C++26 std::execution 高质量 Sender 诊断 + constexpr 异常（提案 P3557R3，Eric Niebler，已纳入 C++26）。这是 std::execution（Sender/Receiver）框架在 C++26 中最重要的诊断改进之一，结合了 P3068（constexpr 异常）带来的能力，让类型错误诊断从“海量模板垃圾”变成清晰、简洁、有意义的错误信息。1. 核心问题（以前的痛点）Sender 是表达式树，类型检查（completion signatures 计算）涉及复杂的元编程。用户写错一个 then、let_value 或 lambda 参数时：

- 错误可能深埋在嵌套 Sender 内部。
- 编译器吐出几千行难以理解的模板实例化栈。
- 诊断位置远离用户代码，难以定位。
- C++26 的解决方案利用 C++26 constexpr 异常（常量求值中可以 throw，未捕获时产生编译错误，并打印异常的 .what() 和类型信息）：

- 将 get_completion_signatures 的计算改为 consteval 函数。
- 类型错误时直接 throw 一个携带诊断信息的异常。
- 编译器在常量求值失败时，精确报告异常类型和内容，而不是底层模板失败细节。
- 主要设计变更（P3557）

- std::execution::get_completion_signatures 改为 consteval 函数模板（无运行时参数）：

  cpp

  ```cpp
  template <class Sndr, class... Env>
  consteval auto get_completion_signatures();   // 返回 completion_signatures<...>
  ```

- Sender 自定义 completion signatures 的方式改为 static consteval 函数（模板参数传入类型）：

  cpp

  ```cpp
  struct my_sender {
      template<class Self, class... Env>
      static consteval auto get_completion_signatures() {
          if constexpr (/* 类型检查失败 */) {
              throw some_diagnostic_error("清晰的错误描述");  // 关键！
          }
          return completion_signatures<set_value_t(int)>{};
      }
  };
  ```

- sender_in<Sndr, Env...> 概念现在要求 get_completion_signatures 是合法的常量表达式。

- 引入 dependent_sender 概念（完成签名依赖环境的 Sender）。

- 算法内部（如 then、when_all 等）在类型不匹配时抛出带上下文的异常（算法名、函数、参数等）。

- 实际效果对比（来自提案）错误示例：just(42) | then([](){})（lambda 不接受 int 参数）C++26（使用 constexpr 异常）：

- 错误位置直接指向用户代码行。
- 清晰说明：“The function passed to std::execution::then is not callable with the values sent by the predecessor sender.”
- 无论嵌套多深，诊断信息都保持简洁。

之前（无此提案）：

- 几千行模板错误，栈被截断，难以定位根本原因。
- 益处

- 用户体验大幅提升：诊断更接近源头、更可读。
- 库作者更轻松：写 Sender 算法时可以用 throw 报告领域特定错误，而非复杂的 SFINAE/requires 技巧。
- 与 constexpr 异常天然结合：常量求值失败时，编译器会打印异常的 what() 信息。
- 简化了 basic_sender、transform_sender 等内部机制。
- 相关特性

- 依赖 P3068（constexpr 异常）：常量求值中 throw / catch 成为可能，未捕获 throw 产生编译错误 + 良好诊断。
- 与 P3164（非依赖 Sender 早期类型检查）协同工作，把诊断时机提前到 Sender 构造时。
- 适用于 std::execution::task、async_scope 等高层抽象。

总结P3557 是 std::execution 在 C++26 中“可用性”的杀手级改进。它把 Sender/Receiver 这个强大但曾经“诊断地狱”的框架，变成了现代、友好、可生产使用的异步/并行工具。这体现了 C++26 的一个重要哲学：不仅要提供强大机制，还要让错误发生时能给出高质量、可理解的反馈。想看更多具体例子（let_value 类型错误、自定义 Sender 抛异常、完整诊断输出）吗？