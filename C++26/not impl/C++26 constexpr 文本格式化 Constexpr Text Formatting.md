**C++26 `constexpr` 文本格式化（Constexpr Text Formatting）** 解释（主要提案 **P3391R2**）。

这是 C++26 中 `std::format` 系列函数的重要提升，特性测试宏为 `__cpp_lib_constexpr_format`。

### 背景

C++20 引入了 `std::format`（基于 `{fmt}` 库），极大改善了格式化体验。但它**不是 `constexpr`**，导致在编译期（如 `static_assert`、`constexpr` 变量初始化、模板元编程）中无法使用。

C++23 已经让 `std::to_chars`（整数）变为 `constexpr`，为格式化打下基础。C++26 则把整个格式化框架“constexpr 化”。

### 主要变化（P3391）

- 将以下函数及其宽字符版本标记为 `constexpr`：
  - `std::format`
  - `std::vformat`
  - `std::format_to` / `std::format_to_n`
  - `std::formatted_size`
- 底层组件也支持 `constexpr`：
  - `std::basic_format_context`
  - `std::basic_format_arg`
  - `std::basic_format_string`
  - 标准格式化器（`formatter`）的 `format` 成员函数等。

**限制**（当前阶段）：
- **不支持** 浮点数（`float`、`double`）和 `chrono` 类型（日期时间）的编译期格式化。
- `L` 本地化（locale）说明符在 `constexpr` 上下文中被禁用（因为 locale 处理通常涉及运行时数据）。
- 依赖于已 `constexpr` 的 `to_chars`（仅整数等基本类型）。

### 实际用途示例

```cpp
#include <format>
#include <cassert>

// 1. static_assert 中直接使用
static_assert(std::format("{} {}", 1, 2) == "1 2");
static_assert(std::format("Hello, {}!", "world") == "Hello, world!");

// 2. 编译期生成消息
static_assert(sizeof(void*) == 8,
    std::format("Expected 64-bit platform, but pointer is {} bytes", sizeof(void*)));

// 3. constexpr 字符串常量
constexpr auto msg = std::format("Error code: {}", 404);
static_assert(msg == "Error code: 404");

// 4. 结合其他 constexpr 特性
constexpr std::string_view generate_log(int level, std::string_view file) {
    return std::format("[{}] {}: {}", level, file, "message");
}
```

### 益处

- **编译期检查**：错误消息、版本信息、配置字符串等可以在编译期生成并验证，大幅提升安全性（`static_assert` 更强大）。
- **零运行时开销**：适合嵌入式、内核、性能敏感场景。
- **与 `static_assert` 完美结合**：P2741R3 让 `static_assert` 支持格式化字符串，进一步放大效果。
- **向后兼容**：运行时行为完全不变，仅新增编译期能力。
- 为未来反射（Reflection）等特性提供更好支持（编译期生成字符串）。

### 实现状态

- GCC（libstdc++）、Clang（libc++）、MSVC 等主流实现已在跟进 P3391。
- `{fmt}` 库早已支持类似 constexpr 实验功能，为标准化提供了基础。

这是一个“让常用高层 API 拥抱 constexpr”的典型 C++26 改进，让文本格式化从运行时工具升级为编译期/运行期两用的强大设施。更多细节可查阅论文 **P3391R2**。

与之前的 `integer_sequence`、`flat_map` 等提案一样，这属于 C++26 “更多 constexpr” 浪潮的一部分。