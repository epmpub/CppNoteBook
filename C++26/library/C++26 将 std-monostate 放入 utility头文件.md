**C++26: 将 `std::monostate` 放入 `<utility>` 头文件**

这是 C++26 中的一个**小型但实用的库改进**，通过提案 **P0472R3** 实现。

### 背景

- `std::monostate` 自 **C++17** 起引入，定义在 `<variant>` 头文件中。
- 它的唯一用途是作为一个**空状态**（unit type / monostate），常用于 `std::variant` 的第一个备选类型，让 `variant` 默认构造时处于“无值”状态，而不需要构造第一个真实类型。

```cpp
// 旧代码（C++17 ~ C++23）
#include <variant>

std::variant<std::monostate, int, std::string> v;  // 默认是 monostate
```

### C++26 的变更

**将 `std::monostate` 移动到 `<utility>` 头文件**（同时在 `<variant>` 中保留别名以保持兼容）。

```cpp
// C++26 推荐写法
#include <utility>        // 现在可以只包含 <utility>

std::monostate ms;        // OK
std::variant<std::monostate, int, double> v;
```

### 为什么要做这个移动？

1. **更合理的归属**：`std::monostate` 是一个**通用工具类型**，与 `std::pair`、`std::tuple`、`std::optional`、`std::any` 等同属“基础工具”，而不是仅服务于 `std::variant`。
2. **减少头文件依赖**：很多场景下你只需要 `monostate`（例如作为占位类型、EBO 辅助、模板元编程等），却不得不包含整个 `<variant>` 头文件，增加了编译时间。
3. **一致性**：类似 `std::in_place`、`std::in_place_index`、`std::in_place_type` 等已放在 `<utility>` 中。

### 实际影响

- **向后兼容**：旧代码（`#include <variant>`）继续正常工作。
- **新代码**：推荐直接 `#include <utility>`。
- **大小与性能**：`std::monostate` 是一个空类（`sizeof == 1`），无运行时开销。

### 示例

```cpp
#include <utility>
#include <variant>
#include <string>
#include <print>

int main() {
    std::variant<std::monostate, int, std::string> v;
    // v 默认持有 monostate
    v.emplace<int>(42);  // 现在持有 int
    if (std::holds_alternative<std::monostate>(v)) {
        std::println("v holds monostate");
    }else if (std::holds_alternative<int>(v)) {
        std::println("v holds int: {}", std::get<int>(v));
    } else if (std::holds_alternative<std::string>(v)) {
        std::println("v holds string: {}", std::get<std::string>(v));
    }
    
    return 0;
}
```

### 总结

这是一个**清理性质**的改进，体现了 C++26 对标准库组织性的持续打磨。虽然功能本身没有变化，但提升了可用性和头文件 hygiene（卫生）。

**提案**：P0472R3  
**cppreference**：`std::monostate` 已标记 C++26 移动到 `<utility>`。

如果你需要了解 `std::monostate` 的其他高级用法（如模板编程中的零大小优化、状态机等），随时告诉我！