#### 错误处理

std::expected<T, E> 是 C++23 引入的一个非常重要的错误处理类型（位于头文件 <expected> 中）。它专门用来表示一个函数要么返回预期的成功值 T，要么返回一个错误 E，是一种类型安全、显式、可组合的错误处理方式。1. 为什么需要 std::expected？传统错误处理方式的痛点：

- 返回 int 错误码 → 容易忽略、语义不清晰
- 使用异常 → 有性能开销、控制流不明显、跨模块难处理
- std::optional<T> → 只能表示“有值或无值”，无法携带为什么失败的信息

std::expected<T, E> 完美解决了这些问题：它强制调用者处理错误，同时能携带丰富的错误信息。2. 基本用法

cpp

```cpp
#include <expected>
#include <string>
#include <iostream>

std::expected<int, std::string> divide(int a, int b)
{
    if (b == 0) {
        return std::unexpected("Division by zero");   // 返回错误
    }
    return a / b;                                     // 返回成功值
}

int main()
{
    auto result = divide(10, 2);

    if (result.has_value()) {          // 或 if (result)
        std::cout << "Success: " << *result << "\n";     // *result 或 result.value()
    } else {
        std::cout << "Error: " << result.error() << "\n";
    }

    // 更简洁的写法（C++23 风格）
    if (result) {
        std::cout << "Result: " << result.value() << "\n";
    } else {
        std::cout << "Failed: " << result.error() << "\n";
    }
}
```

关键点：

- std::unexpected<E>{value} 用于构造错误分支。
- if (result) 或 result.has_value() 判断是否有成功值。
  如果 (result) 或 result.has_value() 判断是否成功。
- 成功时用 *result、result.value() 访问值。
- 失败时用 result.error() 访问错误。
- 更推荐的错误类型

cpp

```cpp
enum class ErrorCode {
    DivisionByZero,
    InvalidInput,
    Overflow,
    NotFound
};

// 使用枚举类作为错误类型（强烈推荐，类型安全）
std::expected<double, ErrorCode> safe_divide(double a, double b)
{
    if (b == 0.0) return std::unexpected(ErrorCode::DivisionByZero);
    if (std::isinf(a / b)) return std::unexpected(ErrorCode::Overflow);
    return a / b;
}
```

你也可以用 std::string、absl::Status、自定义 struct 等作为 E。

4. Monadic 操作（函数式链式调用）—— C++23 的亮点std::expected 支持类似 std::optional 的 monadic 接口，能写出非常优雅的错误处理流水线：

cpp

```cpp
std::expected<std::string, ErrorCode> process_user(std::string_view input)
{
    return parse_id(input)                  // 返回 expected<int, ErrorCode>
           .and_then([](int id) {           // 如果成功，继续执行
               return validate_id(id);
           })
           .and_then([](int valid_id) {
               return fetch_user_from_db(valid_id);
           })
           .transform([](const User& user) { // 纯转换，不会失败
               return user.name();
           })
           .or_else([](ErrorCode e) {        // 统一错误处理
               log_error(e);
               return std::unexpected(e);
           });
}
```

常用 monadic 函数：

- and_then：如果有值，则调用返回 expected 的函数（用于可能失败的操作）
- transform：如果有值，则调用返回普通值的函数（纯映射）
- or_else：如果有错误，则调用处理错误的函数
- transform_error：转换错误类型（比如从底层错误转为高层业务错误）
- 与 Abseil absl::StatusOr<T> 的对比（你之前问过）

| 维度                  | std::expected<T, E>              | absl::StatusOr<T> absl::状态或<T>                            |
| --------------------- | -------------------------------- | ------------------------------------------------------------ |
| 错误类型              | 任意类型 E（灵活）               | 固定为 absl::Status（丰富 Payload） 已修复至 absl::Status （富有效载荷） |
| 标准化程度            | C++23 标准                       | Abseil 库                                                    |
| Monadic 操作 一元操作 | 支持（and_then、transform 等）   | 不直接支持（需手动处理）                                     |
| 错误丰富度            | 依赖你设计的 E                   | 天然支持 message() + 多个 Payload 天然支持 message() + 多个 Payload |
| 适用场景              | 轻量、自定义错误类型、函数式风格 | 大型系统、gRPC、需要结构化丰富错误信息                       |

建议：

- 新项目、想用标准库、错误类型简单 → 优先用 std::expected
- 已大量使用 Abseil、需要丰富上下文（请求ID、堆栈、JSON细节）→ 继续用 absl::StatusOr
- 可以混合使用：底层返回 std::expected，上层转为 absl::StatusOr
- 最佳实践

- 函数返回 std::expected 时，加上 [[nodiscard]]（虽然 expected 本身已有一定提示）。
- 错误类型 E 建议用 enum class 或自定义 struct（避免直接用 std::string 导致错误信息难以统一处理）。
- 错误传播时尽量保持 E 类型一致。
- 不要在极热路径（每秒百万次）的循环里滥用（有少量开销）。
- 结合 std::format、absl::StrCat 构建清晰错误消息。

你目前是想在普通函数中使用，还是想和之前的 absl::Status / absl::btree_map 等结合使用？
或者你想看一个更完整的实际例子（比如文件读取、数据库查询、用户注册流程）？告诉我你的具体场景，我可以给你针对性的代码。