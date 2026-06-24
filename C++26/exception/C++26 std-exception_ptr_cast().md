C++26 std::exception_ptr_cast()

（提案 P2927R2，已被纳入 C++26）。这是对 std::exception_ptr 的重要增强，允许不通过 rethrow_exception + catch 块直接安全地检查（cast）其中存储的异常对象。1. 为什么需要这个功能？以前要检查 exception_ptr 里是什么异常，必须这样做：

cpp

```cpp
bool should_retry(const std::exception_ptr& ep) {
    try {
        std::rethrow_exception(ep);
    } catch (const std::system_error& e) {
        return e.code() == std::errc::device_or_resource_busy;
    } catch (...) {
        return false;
    }
}
```

缺点：

- 性能差（尤其是异步代码中频繁检查时，可慢 100 倍）。
- 引入异常处理开销和栈展开。
- 代码冗长、不优雅。
- 在高性能、异步、并行环境中（如 Sender/Receiver、future、task 等）非常不便。

exception_ptr_cast 提供了**零成本（或极低成本）**的类型安全检查方式。2. 接口

cpp

```cpp
#include <exception>

template <class E>
const E* exception_ptr_cast(const std::exception_ptr& p) noexcept;
```

- 模板参数 E：必须是 cv-unqualified 的完整对象类型（不能是数组、指针、指针成员等）。
- 返回：如果 p 非空，且存储的异常对象能被 const E& 类型的 catch 处理器匹配，则返回指向该异常对象的 const E*；否则返回 nullptr。
- 行为类似 dynamic_cast，但专为 exception_ptr 优化。
- noexcept，永远不抛出异常。
- 使用示例

cpp

```cpp
#include <exception>
#include <iostream>
#include <stdexcept>

struct MyError : std::runtime_error {
    MyError() : std::runtime_error("My custom error") {}
    int error_code = 42;
};

int main() {
    std::exception_ptr ep = std::make_exception_ptr(MyError{});

    // 安全检查
    if (auto* e = std::exception_ptr_cast<MyError>(ep)) {
        std::cout << "MyError: " << e->what() << ", code=" << e->error_code << '\n';
    }

    if (auto* e = std::exception_ptr_cast<std::runtime_error>(ep)) {
        std::cout << "runtime_error: " << e->what() << '\n';
    }

    if (auto* e = std::exception_ptr_cast<std::logic_error>(ep)) {
        std::cout << "logic_error (不会进入)\n";
    } else {
        std::cout << "不是 logic_error\n";
    }

    // 基类也能匹配
    if (auto* e = std::exception_ptr_cast<std::exception>(ep)) {
        std::cout << "base exception: " << e->what() << '\n';
    }
}
```

与多继承的处理：支持正确的多继承和虚继承调整（实现依赖 ABI，但标准保证行为正确）。4. 设计要点

- 始终返回 const E*（强调 const-correctness）。如果需要修改，可显式 const_cast。

- 严格限制：不允许指针类型（如 E = int*），因为 catch 指针时可能绑定到临时对象。

- 性能：实现通常利用 ABI 内部信息（vtable、type_info 等）直接查询，无需实际抛出/捕获。

- 特性测试宏：

  cpp

  ```cpp
  #ifdef __cpp_lib_exception_ptr_cast
  // 值通常为 202403L
  #endif
  ```

- 与现有机制对比

| 方式                       | 性能 | 代码简洁度 | 异常安全性 | 适用场景       |
| -------------------------- | ---- | ---------- | ---------- | -------------- |
| rethrow_exception + catch  | 较差 | 一般       | 安全       | 简单场景       |
| exception_ptr_cast (C++26) | 优秀 | 高         | 安全       | 异步、高频检查 |
| Folly 等第三方实现         | 优秀 | 高         | 安全       | 已有项目       |

总结std::exception_ptr_cast 是 C++26 在异常处理可用性与性能上的重要改进。它让 exception_ptr 从“只能通过重抛检查”的不透明句柄，变成了可以高效查询的类型，让异步代码、错误处理框架、模式匹配（未来）等场景更加高效和现代。特别适合与 std::expected、std::execution::task、async_scope 等 C++26 新特性配合使用。需要 exception_ptr_cast 与模式匹配结合的例子，或性能对比细节吗？