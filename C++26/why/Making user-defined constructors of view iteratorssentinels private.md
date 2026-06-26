**DR20: Making user-defined constructors of view iterators/sentinels private**（P3059R2）解释。

这是 C++26 中的一个**Defect Report (DR)**，建议**回溯应用到 C++20**（Ranges 引入之时）。提案作者：Hewill Kang。

### 背景问题

C++20 `std::ranges` 中的许多 **view**（如 `filter_view`、`transform_view`、`take_view`、`iota_view` 等）的**迭代器（iterator）** 和**哨兵（sentinel）** 类型，暴露了**用户定义的构造函数**（user-provided constructors）。

**具体问题**：
- 这些构造函数原本设计用于**内部实现**（让 view 能构造自己的 iterator/sentinel）。
- 但由于它们是 **public** 的，用户代码可以直接调用，导致：
  - **不一致性**：不同 view 的 iterator/sentinel 构造方式不统一（有些是 aggregate，有些有 public ctor）。
  - **意外使用**：用户可能直接构造 `some_view::iterator`（绕过 view 接口），导致**未定义行为**或违反 view 的不变式（invariants）。
  - **实现 bug**：如 libstdc++ 中出现的副作用。
  - **API 污染**：暴露了不应该对外可见的实现细节，增加了维护负担和误用风险。

例如（简化）：
```cpp
// C++20/23 可能允许
auto it = std::ranges::filter_view<...>::iterator{...};  // 不应该公开
```

这违反了“view 的 iterator/sentinel 应主要通过 view 接口间接使用”的设计原则。

### 提案解决方案（P3059）

- 将视图迭代器和哨兵中**所有用户定义的构造函数**改为 **private**（或 protected，如果需要子类/友元访问）。
- 同时保留/调整必要的 **aggregate 初始化** 或 **默认构造函数**（如果适用）。
- 为内部构造提供适当的**友元声明**或工厂函数，让 view 自身仍能正常构造 iterator/sentinel。
- 清理相关规范，确保一致性。

**核心变化**：
- 对用户代码：**无法直接构造**这些 iterator/sentinel（编译错误）。
- 对 view 内部：构造行为不变。
- 少数需要默认构造的场景仍支持（例如 `iota_view` 的 iterator）。

### 效果与益处

- **安全性**：防止用户误用或滥用内部类型，减少 UB 风险。
- **一致性**：所有标准 views 的 iterator/sentinel 遵循相同规则。
- **封装性**：更好地隐藏实现细节，符合 Ranges 设计哲学（用户应通过 `views::xxx` 和 range 接口交互）。
- **向后兼容**：绝大多数用户代码不受影响（因为很少有人直接构造这些内部类型）。少数依赖旧行为的代码需调整。
- 作为 DR 回溯到 C++20：现有实现需修复，未来代码更规范。

### 实际影响示例

```cpp
// 之前（可能允许，但危险）
std::ranges::transform_view<V, F>::iterator it = /* 直接构造 */;

// 之后（C++26 + DR）
std::ranges::transform_view<V, F>::iterator it;  // 如果支持默认构造则 OK，否则错误
// 正确方式始终是：
auto v = std::views::transform(base, f);
auto it = v.begin();   // 通过 view 接口
```

Cppreference 已将此标记为 **DR20**，并在相关 views 的 iterator/sentinel 页面注明构造函数可见性变化。主流实现（如 libc++、libstdc++、MSVC）正在/已经跟进。

这是一个典型的“**加强封装、修复历史设计疏漏**”的 C++ Ranges 清理提案，让 Ranges 接口更健壮、更难被误用。完整细节可查阅论文 **P3059R2**。