**DR11（P3016R6）——解决 `std::valarray` 和 `std::initializer_list` 中 `begin`/`end` 等函数的不一致性**解释。

这是 C++26 中的一个重要库缺陷报告（Defect Report），已作为 DR 回溯应用到 C++11（部分实现有差异）。

### 主要问题（不一致性）

1. **`std::valarray` 的 `begin`/`end` 支持不完整**：
   - `valarray` 提供了非成员（non-member）`std::begin`/`std::end` 重载（在 `<valarray>` 中），用于支持 range-based for。
   - 但 `std::cbegin(v)`、`std::rbegin(v)` 等常因包含顺序、ADL（Argument-Dependent Lookup）和模板定义时机问题而编译失败（尤其在 libc++ 和 MSVC 上）。
   - `valarray` 缺少成员 `begin()`/`end()`，导致与现代 Ranges/迭代器接口不一致。

2. **Braced initializer list（`{1,2,3}`）的 `std::begin` 支持有害**：
   - `std::begin({1,2,3})` 能编译（通过 `<initializer_list>` 中的特殊重载），但返回的迭代器会 **dangle**（悬垂），且 `begin` 和 `end` 可能指向不同临时数组，无法形成有效 range。
   - `std::cbegin({1,2,3})` 等却无法编译，行为不一致且危险。

这些问题源于 C++0x 时代 range-for 设计的遗留（当时对 braced-init-list 和成员 vs 非成员函数的处理不完善）。

### 提案的解决方案（P3016）

- **为 `std::valarray` 添加成员 `begin()`、`end()`、`cbegin()` 等**（类似普通容器/Range），并移除或调整非成员重载，使 `std::begin`/`cbegin`/`rbegin` 等一致可用。

- **为 `std::initializer_list<T>` 添加成员**：
  - `.data()`（返回 `const T*`）
  - `.empty()`
  - 这使得 `std::data(il)`、`std::empty(il)` 等泛型函数能通过主模板工作（不再需要特殊重载）。

- **移除 `std::initializer_list` 的非必要非成员 `begin`/`end` 重载**（保留成员函数即可）。
  - 直接对 braced-init-list 调用 `std::begin({1,2,3})` 变为 ill-formed（这是好事，避免 dangling）。
  - 对 `std::initializer_list` 对象本身的 `std::begin(il)` 仍正常工作（需包含 `<iterator>`）。

- 清理库中多处使用 `begin()+size()` 的地方，改为更自然的 `data()+size()`。

- 添加特性测试宏：`__cpp_lib_initializer_list` 和 `__cpp_lib_valarray`。

### 效果与益处

- **一致性**：`valarray` 现在更像一个 Range，支持完整的 `std::begin`、`ranges::begin` 等。
- **安全性**：消除 braced-init-list 上危险的 `begin` 使用。
- **便利性**：`initializer_list` 现在有 `.data()` 和 `.empty()`，与 `span`、`string_view` 等一致。
- **向后兼容**：大多数代码不受影响；少数依赖旧行为的代码需包含 `<iterator>`。
- 同时解决了多个 LWG issues（如 LWG3624、LWG3625 等）。

### 实际代码变化示例

```cpp
// 之前（可能失败或危险）
std::valarray<int> v = {1,2,3};
auto it = std::cbegin(v);          // 可能错误

auto dangling = std::begin({1,2,3}); // 危险，dangling

// 之后（C++26）
std::valarray<int> v = {1,2,3};
auto it = std::cbegin(v);          // OK

std::initializer_list<int> il = {1,2,3};
const int* p = il.data();          // 新增成员
if (!il.empty()) { ... }

std::begin({1,2,3});               // 编译错误（好！）
```

Cppreference 已更新相关页面，标记为 DR11。

这是一个典型的“清理历史遗留、提升一致性和安全性”的 C++ 演进例子，由 Arthur O'Dwyer 主笔。更多细节可查阅 P3016R6 论文。