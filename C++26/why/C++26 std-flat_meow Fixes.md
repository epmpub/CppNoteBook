**`std::flat_meow` Fixes（P3567R2）——修复 `std::flat_map` / `flat_set` 等容器的缺陷**解释。

“`flat_meow`” 是提案作者（Hui Xie、Louis Dionne、Arthur O’Dwyer 等）使用的**幽默占位符**，指代 `flat_map`、`flat_multimap`、`flat_set`、`flat_multiset` 这组 C++23 引入的扁平（sorted vector-based）关联容器。

这是 C++26 的一个小修复提案（B2 类改进），建议作为 **Defect Report (DR)** 回溯应用到 C++23（libc++ 等实现已跟进）。

### 主要修复内容

1. **LWG4000: `insert_range` 的 Effects 规范错误**（`flat_map`）
   - 原来规范有 bug：循环中使用 `const auto& e` 导致无法正确 `move`，且未使用 `value_type` 显式确保是 `pair`。
   - 修复：改为 `ranges::for_each` + 显式 `value_type`，正确处理 move，并与规范其他部分保持一致。

2. **`swap` 改为条件 `noexcept`**
   - 原来是无条件 `noexcept`，但底层容器（如 `vector`）的 `swap` 可能抛出（取决于 allocator 等）。
   - 问题：要么 silently 清空容器（坏），要么 `std::terminate`（更坏）。
   - 修复：`noexcept(is_nothrow_swappable_v<...>)`，允许异常传播（同时恢复不变性）。这与普通容器行为一致。

3. **缺失的 `insert_range(sorted_unique / sorted_equivalent, rg)` 重载**
   - 其他插入接口都有 “已排序” 版本（提示实现可做线性合并），但 `insert_range` 缺少对应重载。
   - 修复：为 `flat_map`/`flat_set` 添加 `sorted_unique` 版本，为 multi 版本添加 `sorted_equivalent` 版本。
   - 已排序版本复杂度为线性（in-place merge），大幅提升性能。

4. **`flat_set::insert_range` 避免不必要拷贝**
   - 原来规范强制 `const auto&` 导致拷贝。
   - 修复：使用 `auto&&` + `std::forward`，允许 move（类似其他容器）。

5. **特殊成员函数（构造函数、赋值等）显式规范**
   - 原来未明确指定，导致依赖 `=default` 的实现可能不符合要求（例如异常安全性、constexpr 等）。
   - 修复：明确规范这些函数的行为，使实现更一致。

### 效果与益处

- **正确性**：修复 LWG issue 和规范不一致。
- **性能**：已排序插入现在能高效利用线性合并；`swap` 更安全。
- **一致性**：与 `std::map` / `std::set` 等传统容器，以及 Ranges 接口对齐。
- **向后兼容**：主要是修复，破坏性极小（少数依赖旧 `noexcept` 的代码可能受影响，但这是正确的）。
- 引入/更新特性测试宏（`__cpp_lib_flat_map`、`__cpp_lib_flat_set` 等）。

### 实际影响

```cpp
std::flat_map<Key, Value> m;
auto sorted_range = /* 已排序的 range */;
m.insert_range(std::ranges::sorted_unique, sorted_range);  // 新增，支持高效线性插入

std::flat_set<T> s;
s.swap(other);  // 现在 noexcept 更精确
```

Cppreference 的 C++26 页面已标记此变更。libc++、MSVC 等实现已支持（部分作为 DR 回溯）。

这是一个典型的“实现经验驱动的清理提案”——libc++ 在实现 `flat_*` 时发现的问题，提出来让标准更健壮。完整细节见 P3567R2 论文（基于 P2767R2 的子集，避免争议设计变更）。