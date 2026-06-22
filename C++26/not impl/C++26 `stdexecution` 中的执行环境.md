**C++26 `std::execution` 中的执行环境（Execution Environment）工具**

C++26 的 **Execution Control Library**（`<execution>` 头文件，提案 **P2300**）引入了 **Execution Environment**（执行环境）的概念。它是一种**可查询的（queryable）**键值对集合，用于在异步操作（senders/receivers）中传递上下文信息，例如：

- 当前调度器（`get_scheduler`）
- 停止令牌（`get_stop_token`）
- 分配器（`get_allocator`）
- 域（domain）、前向进度保证等

### 主要工具（P3325R5）

提案 **P3325**（*A Utility for Creating Execution Environments*）提供了两个简单实用的工具，用于**轻松构造执行环境**：

#### 1. `std::execution::prop<Query, Value>`

- **作用**：用一个查询（query）和一个值构建一个**单一属性**的环境。
- **用途**：为特定属性提供自定义值。

```cpp
#include <execution>

auto my_env = std::execution::prop{std::execution::get_scheduler, my_scheduler};
```

#### 2. `std::execution::env<Env1, Env2, ...>`

- **作用**：将多个可查询对象（或 `prop`）**聚合**成一个复合执行环境。
- 支持**覆盖**（后面的属性会覆盖前面的同名属性）。

```cpp
auto combined_env = std::execution::env{
    std::execution::prop{std::execution::get_allocator, my_alloc},
    std::execution::prop{std::execution::get_stop_token, my_token},
    other_env
};
```

### 核心函数

- **`std::execution::get_env(obj)`**：从 receiver 或其他对象获取其关联的执行环境（自定义点对象）。
- **`std::execution::read_env<Query>()`**：创建一个 sender，用于从当前环境中读取特定查询的值。

### 典型使用场景

```cpp
using namespace std::execution;

auto my_scheduler = /* some thread pool scheduler */;

auto work = just(42)
          | then([](int v) { return v * 2; })
          | on(my_scheduler);   // 在特定调度器上执行

// 自定义环境（传递额外上下文）
auto custom_env = prop{get_allocator, my_custom_allocator};

// 在 receiver 中使用环境
struct MyReceiver {
    /* ... */
    auto get_env() const noexcept {
        return env{
            prop{get_scheduler, my_scheduler},
            prop{get_allocator, my_allocator}
        };
    }
};
```

### 为什么需要这些工具？

- 执行环境是 **extensible**（可扩展）的，允许库和用户在不修改 sender/receiver 接口的情况下传递额外上下文。
- `prop` 和 `env` 让构造复杂环境变得简洁、直观。
- 这些工具是**轻量级**的，通常是纯编译期构造，几乎无运行时开销。

### Feature Test Macro

```cpp
#if __cpp_lib_execution >= 2024XXL  // 具体值视实现而定
```

**参考资料**：
- cppreference: [Execution control library](https://en.cppreference.com/w/cpp/execution)（Environments 部分）
- 提案: [P3325R5](https://wg21.link/P3325)

这个工具是 C++26 **std::execution**（senders/receivers 框架）的重要辅助设施，让异步编程中的上下文传递更加灵活和类型安全。

如果你需要完整示例（结合 `stdexec` 参考实现、scheduler 创建、完整 pipeline 等），或者想了解如何在 GCC/Clang 上实验这些特性，请告诉我！