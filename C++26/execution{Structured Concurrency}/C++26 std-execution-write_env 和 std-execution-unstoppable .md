C++26 std::execution::write_env 和 std::execution::unstoppable Sender Adaptors（提案 P3284R4，已被纳入 C++26）。

这两个适配器是 std::execution（Sender/Receiver 框架）中对执行环境（execution environment） 的重要补充，主要用于隐式参数传递和精细的取消（stop）控制。它们最初是作为其他提案（如 on 算法改进）的辅助工具，后来因实用性独立标准化。 

open-std.org

1. 执行环境（Execution Environment）回顾Receiver 关联一个可查询的键值存储（queryable object），用于在 Sender 树中向下传递隐式上下文，如：

- 当前调度器（get_scheduler）
- 停止令牌（get_stop_token）
- 分配器（get_allocator）
- 其他领域特定查询

read_env 用于读取，write_env 用于写入/覆盖。2. write_env —— 修改执行环境

cpp

```cpp
namespace std::execution {
    inline constexpr unspecified write_env{};  // CPO
}
```

- 作用：接受一个 Sender 和一个 queryable 环境对象，返回一个新 Sender。新 Sender 在连接 Receiver 时，会把提供的环境叠加（join）到 Receiver 的原始环境上。
- 查询优先级：新环境优先（先查 env，查不到再查原始 get_env(rcvr)）。
- 不可自定义（uncustomizable），但支持管道风格（pipeable）。
- 主要用途：向子操作注入上下文（如分配器、特定调度器等），简化 let_* 算法和 on 算法的规范。

示例（注入自定义分配器）：

cpp

```cpp
namespace ex = std::execution;

// 自定义 with_allocator 适配器（用户可自行封装）
auto work = third_party::make_async_work()
          | with_allocator(custom_alloc);  // 内部使用 write_env

// 第三方代码可能这样读取：
ex::let_value(ex::read_env(std::get_allocator),
    [](auto alloc) { /* 使用 alloc 创建 vector 等 */ });
```

3. unstoppable —— 忽略外部停止请求

cpp

```cpp
namespace std::execution {
    inline constexpr unspecified unstoppable{};  // CPO
}
```

- 作用：使 Sender 不再响应外部停止请求（stop token 被替换为 never_stop_token）。
- 实现：本质上是 write_env(sndr, prop(get_stop_token, never_stop_token{})) 的便捷包装。
- 关键场景：
  - 清理/恢复操作（必须完成，不能被取消，否则可能破坏程序不变量）。
  - “async RAII” 中的 finally/cleanup 步骤。
  - 必须成功完成的临界工作。

示例（与 finally 配合恢复不变量）：

cpp

```cpp
namespace ex = std::execution;

auto safely_munge_data() {
    return ex::let_value([](auto&... values) {
        return break_invariants(values...)
             | ex::then(do_munge)
             | ex::finally(ex::unstoppable(restore_invariants(values...)));
    });
}
```

即使外部请求停止，restore_invariants 也会执行（因为 unstoppable）。4. 与其他特性的关系

- 与 read_env：一对互补操作（读 vs 写）。
- 与 on / starts_on / continues_on：write_env 简化了这些调度相关算法的内部实现。
- 与 async_scope / task：在结构化并发中，可用 write_env 传递作用域特定上下文，用 unstoppable 保护清理逻辑。
- 与 finally（早期提案中）：unstoppable 常用于确保清理 Sender 不被取消。
- 设计要点

- 零运行时开销（在大多数实现中）：环境合并是编译期/轻量包装。
- 管道友好：支持 sndr | ex::write_env(my_env) 和 ex::write_env(my_env)（部分应用）。
- 类型安全：环境是 queryable 对象，通常用 prop(query_tag, value) 构造。
- 与早期工作草稿：早期版本中是 exposition-only（内部用），C++26 正式暴露给用户。

总结

- write_env：Sender 版的“with 上下文”或“注入参数”，极大提升了隐式上下文传递的表达力，尤其适合库作者和第三方 Sender。
- unstoppable：提供“必须完成”的保证，是构建可靠异步 RAII、清理逻辑和临界恢复操作的关键工具。

这两个适配器让 std::execution 在上下文管理和取消语义上更加完备和实用，是 C++26 异步编程基础设施的重要补充。需要 write_env 在自定义 Sender 中的实现细节、更多完整示例，还是与 let_value / on 的结合使用场景？