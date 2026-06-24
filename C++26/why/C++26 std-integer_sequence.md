**`std::integer_sequence` 在结构化绑定（Structured Bindings）和扩展语句（Expansion Statements）中的可用性改进（P1789R3）** 解释。

这是 C++26 中的一个小型库提案（Library-only），已采纳，引入特性测试宏 `__cpp_lib_integer_sequence`。

### 背景问题

C++26 引入了两项强大语言特性：
- **结构化绑定引入 pack**（P1061）：`auto [...idx] = something;` —— 允许将可分解对象直接解包为参数包。
- **扩展语句**（`template for` / Expansion Statements，P1306）：允许在编译期对 pack 或可结构化绑定类型进行“for 循环”式展开。

然而，`std::integer_sequence<T, Vals...>`（以及 `std::index_sequence`）**之前无法直接用于这些特性**：
- 它没有实现 tuple protocol（`std::tuple_size`、`std::tuple_element`、`std::get<N>`）。
- 开发者只能通过笨拙的 lambda 技巧绕道：
  ```cpp
  []<std::size_t ...I>(std::index_sequence<I...>) {
      // 使用 I... pack
  }(std::make_index_sequence<N>{});
  ```
  这引入了不必要的嵌套、作用域问题，且代码可读性差，尤其在反射（reflection）场景中问题突出。

### 提案解决方案（P1789）

为 `std::integer_sequence<T, Vals...>` 添加 **tuple protocol 支持**：

- `std::tuple_size` 特化：返回 `sizeof...(Vals)`。
- `std::tuple_element<I, integer_sequence<T, Vals...>>` 特化：返回第 I 个值的类型 `T`。
- `std::get<I>(integer_sequence<T, Vals...>)` 重载：返回第 I 个编译期常量值。

**核心效果**：`std::integer_sequence` 现在可以像 `std::tuple`、`std::array` 或聚合类型一样被结构化绑定，从而无缝支持 pack 引入和扩展语句。

### 示例

```cpp
// C++26 新写法

// 1. 直接引入 index pack（结构化绑定）
constexpr auto [...Idx] = std::make_index_sequence<5>{};
// Idx... 是 0,1,2,3,4 的 pack

// 2. 在扩展语句中使用（template for）
template for (constexpr std::size_t I : std::make_index_sequence<10>{}) {
    // I 是编译期常量，每次循环取不同值
    std::cout << I << ' ';
}

// 3. 结合 tuple 比较（更简洁）
template<class... Ts, class... Us>
constexpr bool operator==(const std::tuple<Ts...>& lhs, const std::tuple<Us...>& rhs) 
    requires (sizeof...(Ts) == sizeof...(Us))
{
    constexpr auto [...Idx] = std::index_sequence_for<Ts...>{};
    return ((std::get<Idx>(lhs) == std::get<Idx>(rhs)) && ...);  // fold
}

// 或使用扩展语句（避免 fold 的某些限制）
template for (constexpr std::size_t I : std::index_sequence_for<Ts...>{}) {
    if (std::get<I>(lhs) != std::get<I>(rhs)) return false;
}
return true;
```

### 益处

- **简洁性**：消除大量 lambda  boilerplate，提升可读性和可维护性。
- **与新特性协同**：使 `template for` 和 pack-structured-binding 真正实用，尤其在元编程、反射、constexpr 代码中。
- **零开销**：纯库实现，无运行时成本。
- **向后兼容**：不改变现有代码，仅新增支持。
- 实现已在 libstdc++、libc++ 等跟进。

这是一个典型的“让已有库类型拥抱新语言特性”的改进提案，让 `integer_sequence` 这个老工具在新 C++26 时代焕发新生。完整细节见 P1789R3。