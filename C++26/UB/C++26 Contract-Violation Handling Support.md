**C++26 Contract-Violation Handling Support**

 是 **Contracts** 特性（提案 **P2900R14**）的重要组成部分，定义在新的头文件 `<contracts>` 中。

### 核心机制

当合约断言（precondition、postcondition、assert）**违反**时，程序会调用**合约违反处理器**（Contract Violation Handler）。

#### 1. 全局处理器函数（用户可替换）

```cpp
void handle_contract_violation(std::contracts::contract_violation info);  // 全局 ::handle_contract_violation
```

- **默认处理器**（implementation-provided）：通常打印诊断信息（如位置、谓词、语义等）并终止程序。
- **是否可替换**：实现定义（implementation-defined）。大多数主流实现允许用户在链接时提供自己的版本（类似替换 `operator new`）。
- 可以是 `noexcept` 或抛出异常（取决于语义）。

#### 2. `std::contracts::contract_violation` 类

这是传递给处理器的关键信息对象，包含：

- `location()`：违反发生的位置（`std::source_location`）。
- `kind()`：合约类型（precondition / postcondition / assert）。
- `semantic()`：当前评估语义（observe / enforce / quick-enforce）。
- `description()`：可选的断言描述字符串。
- `detection_mode()`：违反检测模式（正常、异常等）。

```cpp
namespace std::contracts {
    class contract_violation {
    public:
        std::source_location location() const noexcept;
        contract_kind kind() const noexcept;
        contract_semantic semantic() const noexcept;
        std::string_view description() const noexcept;
        // ...
    };
}
```

### 评估语义（Evaluation Semantics）与处理器交互

| 语义            | 是否调用处理器 | 处理器返回后的行为       | 典型用途              |
| --------------- | -------------- | ------------------------ | --------------------- |
| `ignore`        | 否             | 继续执行（无检查）       | Release 优化构建      |
| `observe`       | 是             | 继续正常执行             | 调试 / 监控（不终止） |
| `enforce`       | 是             | 终止程序（terminate）    | 默认安全模式          |
| `quick-enforce` | 否             | 立即终止（不调用处理器） | 最高性能的安全检查    |

### 库支持函数

- **`std::contracts::invoke_default_contract_violation_handler(const contract_violation&)`**：显式调用默认处理器（在自定义处理器中非常有用）。

### 用户自定义处理器示例

```cpp
#include <contracts>
#include <iostream>
#include <cstdlib>

void ::handle_contract_violation(const std::contracts::contract_violation& v) {
    std::cerr << "CONTRACT VIOLATION!\n"
              << "Location: " << v.location().file_name() << ":" << v.location().line() << "\n"
              << "Kind: " << /* ... */ << "\n"
              << "Description: " << v.description() << std::endl;

    // 自定义行为：
    // 1. 记录日志 / 发送监控
    // 2. 抛出异常（仅在 observe/enforce 下有效）
    // 3. 调用默认处理器
    std::contracts::invoke_default_contract_violation_handler(v);

    // 或直接终止
    std::abort();
}
```

### 重要注意事项

- **链接时替换**：自定义处理器必须在链接阶段提供（不能是动态注册）。
- **抛出异常**：用户处理器可以抛出异常，这在 `observe` / `enforce` 语义下允许栈展开（但 `quick-enforce` 不调用处理器）。
- **Observable Checkpoint**：合约违反与 `std::observable_checkpoint()` 配合使用，限制 UB 时间旅行优化。
- **编译期**：编译期违反通常产生警告。

### Feature Test Macro

```cpp
#if __cpp_lib_contracts >= 2025XXL   // 具体值以实现为准
#include <contracts>
#endif
```

**提案**：P2900R14（Contracts for C++）  
**cppreference**：[`std::contracts::contract_violation`](https://en.cppreference.com/w/cpp/contract/contract_violation)

这是 C++26 **安全性与可观测性** 提升的核心设施之一，显著改善了传统 `assert()` 的局限性（更灵活、可配置、携带丰富上下文）。需要自定义处理器完整示例、与 Contracts 断言结合的代码，还是不同语义的对比？随时告诉我！