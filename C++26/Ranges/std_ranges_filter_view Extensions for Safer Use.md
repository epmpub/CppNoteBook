**`std::ranges::filter_view` Extensions for Safer Use（P3725R3）** 解释。

这是 C++26 中的一个**Defect Report (DR)**，建议**回溯应用到 C++20**（Ranges 引入之时）。作者：Nicolai Josuttis。

### 背景问题

`std::ranges::views::filter`（`filter_view`）是 C++20 中最常用但也**最容易出错**的 range adaptor 之一。常见痛点包括：

1. **UB（未定义行为）风险**：
   - 修改过滤后的元素后，如果修改后的值不再满足 predicate，后续迭代可能导致 UB（尤其与 `reverse`、`as_rvalue` 等组合时）。
   - 缓存机制（`begin_` 缓存）与 multi-pass 假设冲突，导致内存覆盖或 core dump。

2. **const 迭代不支持**：
   - `const` 容器上的 `filter` 往往编译失败（因为 `begin()` 非 `const`）。

3. **非直观行为**：
   - 普通程序员难以预测何时安全、何时危险。需要深入了解缓存、引用语义和 predicate 稳定性。

示例（之前危险或错误）：
```cpp
std::vector<std::string> coll{"Amsterdam", "Berlin", "Cologne", "LA"};

auto large = [](const auto& s) { return s.size() > 5; };

// UB / core dump 风险（与 reverse + as_rvalue 组合）
auto sub = coll | std::views::filter(large) | std::views::reverse | std::views::as_rvalue | std::ranges::to<std::vector>();

// const 迭代失败
void constIterate(const auto& r);
constIterate(coll | std::views::filter(large));  // 编译错误
```

### 提案解决方案

引入 **工作区（workaround）**，让安全使用更简单：

- 新增 **`std::views::as_input`**（原 `to_input`，P3828）适配器，将 range 降级为 **input-only**（禁用 multi-pass / 缓存）。
- 为 `filter_view` 添加 **const `begin()` / `end()` 重载**（在底层是 input range 且非 forward range 时）。
- 放松 iterator 修改 UB 的规范描述（更清晰）。

**核心机制**：
- `as_input | filter` 产生一个**只支持单遍（single-pass）**的 view。
- 这个 view **不缓存** `begin()` 位置，行为更安全、可预测。
- `const` 支持自然可用。

### 使用示例（C++26 + DR）

```cpp
// 安全修改元素（之前 UB）
for (auto& m : monsters | std::views::as_input | std::views::filter(dead)) {
    m.bringBackToLive();  // 现在 well-defined
}

// const 迭代
constIterate(coll | std::views::as_input | std::views::filter(large));  // OK

// 危险组合现在编译错误（好！）
auto bad = coll | std::views::as_input | std::views::filter(large)
               | std::views::reverse | std::views::as_rvalue;  // 编译失败
```

不使用 `as_input` 时，旧行为保留（支持 forward/bidirectional 等高阶范畴，但仍需小心 predicate）。

### 益处

- **安全性**：大幅降低常见 UB 场景；危险组合在编译期暴露。
- **易用性**：普通开发者可以用“`as_input + filter`”模式安全组合 pipeline，无需掌握底层缓存细节。
- **向后兼容**：纯扩展，不破坏现有正确代码。
- **教学友好**：更容易向初学者讲解“想安全就加 `as_input`”。
- 与 `views::as_input`（C++26）协同。

Cppreference 已更新 `filter_view` 页面，标记为 DR（P3725R3），并记录 `as_input` 的配合使用。主流实现（libstdc++、libc++、MSVC）正在跟进。

这是一个典型的“**提升 Ranges 安全性、降低学习曲线**”的 C++26 改进，由实际使用痛点驱动。完整细节见论文 **P3725R3**。