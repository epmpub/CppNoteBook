**`std::constant_wrapper` 用于 `std::function_ref` 构造函数** 解释（C++26）。

这是 C++26 中**统一编译期常量传递**的重要改进，主要涉及提案 **P2781**（`std::constant_wrapper`）、**P3948**（“`constant_wrapper` is the only tool needed”）以及 `std::function_ref` 相关变更（P0792 / P2472 / P3740）。

### 背景

- **`std::constant_wrapper<T v>`**（也常写作 `std::cw<v>`）：C++26 引入的**通用编译期常量包装器**。
  - 它是一个 structural type（可作为 NTTP）。
  - 提供 `static constexpr T value = v;` 和 `constexpr operator T() const` 等。
  - 目标是**取代**多个零散的常量包装工具（如 `std::integral_constant`、`std::nontype_t` / `constant_arg_t` 等），实现“一个工具走天下”。

- **`std::function_ref<R(Args...)>`**：C++26 的**非拥有型、可调用引用**（类似 `std::string_view` 对于函数）。
  - 轻量、无分配、 trivially copyable。
  - 支持普通可调用对象、函数指针、成员函数指针等。

早期 `function_ref` 使用 `std::nontype_t<f>`（或 `constant_arg_t`）来支持**编译期已知函数指针 / 成员指针**的构造（避免运行时 type-erasure 开销，生成更优代码）。

### C++26 的统一变更

通过 P3948 等提案，**`std::constant_wrapper` 完全取代了 `nontype_t` / `constant_arg_t`** 在 `function_ref` 中的使用。

`std::function_ref` 的相关构造函数变为：

```cpp
template <auto f>
constexpr function_ref(std::constant_wrapper<f>) noexcept;   // (3)

template <auto f, class U>
constexpr function_ref(std::constant_wrapper<f>, U&& obj) noexcept;

template <auto f, class T>
constexpr function_ref(std::constant_wrapper<f>, cv T* obj) noexcept;
```

**推导指引（deduction guides）** 也相应更新。

### 使用示例

```cpp
#include <functional>

int free_func(int x) { return x * 2; }

struct S {
    int member_func(int x) const { return x + 42; }
    static int static_member(int x) { return x * 10; }
};

void example() {
    // 1. 自由函数
    std::function_ref<int(int)> fr1 = std::constant_wrapper<free_func>{};

    // 2. 静态成员函数
    std::function_ref<int(int)> fr2 = std::constant_wrapper<&S::static_member>{};

    // 3. 成员函数（需绑定对象）
    S obj;
    std::function_ref<int(int)> fr3(std::constant_wrapper<&S::member_func>{}, &obj);

    // 调用
    std::cout << fr1(5) << '\n';   // 10
    std::cout << fr3(10) << '\n';  // 52
}
```

### 益处

- **统一性**：不再需要记住 `nontype`、`constant_arg` 等多个辅助类型。`std::constant_wrapper` 成为唯一工具。
- **性能**：使用编译期已知函数指针时，`function_ref` 可以生成直接调用（无虚调用或 type-erasure 开销），代码更小更快。
- **简洁性**：`std::cw<func>` 或 `std::constant_wrapper<func>{}` 语法清晰。
- **一致性**：与 NTTP、反射、`std::meta` 等 C++26 新特性完美配合。
- **向后兼容**：旧的 `nontype_t` 可能被弃用或作为别名保留。

### 实际意义

这个设计让 `function_ref` 在**回调参数**场景中特别强大（例如事件系统、策略模式、性能敏感的热路径），同时保持了编译期可知的最优性能。

这是一个典型的 C++26 “清理 + 统一” 提案，减少了标准库中的冗余辅助类型，让开发者体验更现代和一致。

更多细节可参考 cppreference 的 C++26 页面、`std::constant_wrapper` 和 `std::function_ref` 文档，以及提案 P3948 / P2781。