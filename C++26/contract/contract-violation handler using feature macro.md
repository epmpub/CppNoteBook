**`implementation-defined replaceability detection of the contract-violation handler using feature macro`**（C++26）解释。

这是 C++26 Contracts 特性（P2900 系列）中的一个小但实用的改进，由 **P3886R0**（AT1-057）提出并采纳，特性测试宏为 **`__cpp_lib_replaceable_contract_violation_handler`**。

### 背景

C++26 引入了**契约（Contracts）** 语言特性，包括 `pre`、`post`、`assert` 等。契约违反时，会根据评估语义（`ignore` / `observe` / `enforce` / `quick-enforce`）进行处理。

其中，**契约违反处理器（contract-violation handler）** 是核心机制：

```cpp
void ::handle_contract_violation(const std::contracts::contract_violation& violation);
```

- 实现提供默认处理器（通常打印信息并终止程序）。
- 用户可以通过**链接时替换（link-time replacement）** 定义自己的版本，实现自定义日志、断点、异常抛出、恢复策略等。

**关键点**：处理器是否**可替换（replaceable）** 是 **implementation-defined**（实现定义的）。某些平台/编译器（出于安全考虑，如防止恶意替换）可能不允许用户替换，而仍符合标准。

之前，代码无法**可移植地检测**当前实现是否支持替换，导致：
- 试图定义替换函数时，可能出现多重定义（ODR 违反）或链接错误（无诊断）。
- 库/可移植代码难以根据实现能力调整行为。

### 解决方案（P3886）

新增**特性测试宏**（Feature Test Macro）：

```cpp
// 在 <contracts> 或 <version> 中
#ifdef __cpp_lib_replaceable_contract_violation_handler
    // 当前实现支持 replaceable handler
    #define __cpp_lib_replaceable_contract_violation_handler 202603L  // 示例值
#endif
```

**效果**：
- 如果宏已定义（且值 >= 指定日期），则 `handle_contract_violation` **可被用户安全替换**。
- 如果宏未定义，则替换行为是 ill-formed（无诊断要求），用户不应提供自己的定义。

### 使用示例

```cpp
#include <contracts>
#include <version>

void handle_contract_violation(const std::contracts::contract_violation& v) {
#ifdef __cpp_lib_replaceable_contract_violation_handler
    // 自定义处理：日志 + 断点 + 终止
    std::cerr << "Contract violation at " << v.location() << ": " 
              << v.comment() << std::endl;
    std::terminate();  // 或其他策略
#else
    // 不可替换时，回退到默认行为（或静态断言/其他处理）
    static_assert(false, "This implementation does not support custom contract handlers");
#endif
}
```

或在库代码中条件编译：

```cpp
#if defined(__cpp_lib_replaceable_contract_violation_handler)
    // 提供替换版本
#else
    // 只依赖默认行为
#endif
```

### 益处

- **可移植性**：代码可以查询实现能力，避免在不支持替换的平台上产生链接/ODR 问题。
- **安全性**：允许某些实现（尤其是高安全环境）禁用替换，同时让用户代码能检测并优雅降级。
- **一致性**：符合 C++ 特性测试宏的常规做法（与其他 `__cpp_lib_*` 宏统一）。
- **向后兼容**：不改变现有语义，仅增加检测手段。

这是一个典型的“让实现定义行为可查询”的 C++26 清理提案，确保 Contracts 特性在不同编译器/平台上更易用。Cppreference 已更新相关文档，主流实现（如 GCC/Clang）通常会定义此宏（支持替换）。更多细节可查阅 P3886R0 及 Contracts 主提案 P2900R14。