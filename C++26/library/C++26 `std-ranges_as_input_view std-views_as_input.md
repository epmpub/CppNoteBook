**C++26 `std::ranges::as_input_view`（`std::views::as_input`）** 是 Ranges 库中的一个**范围适配器**（range adaptor），通过提案 **P3137**（以及后续 P3828 重命名）引入。

### 作用

它将一个**视图**（view）**降级**（downgrade）为：

- **仅 `input_range`**（input-only）
- **非 `common_range`**（non-common）

其他属性（如 `value_type`、`reference_type`、`sized`、`borrowed` 等）尽量保持不变。

```cpp
std::ranges::as_input_view<V>          // class template
std::views::as_input                    // range adaptor object
```

### 为什么需要它？（主要动机）

许多 range adaptors 和算法会**根据输入 range 的 iterator category 选择最强的实现**（例如 `forward_range`、`bidirectional_range`、`random_access_range` 等）。

- **强 category** → 可能产生**更复杂、更慢**的代码（例如缓存、额外状态）。
- **input-only** → 强制使用最**轻量级**的单遍（single-pass）实现，通常性能更好，尤其在：
  - 输入是 `istream_view` 或其他一次性输入。
  - 你想**故意禁用**后续多遍遍历或随机访问优化。
  - 性能敏感的管道（pipeline）中，避免不必要的 overhead。

简单来说：**强制降低范畴以获得更好的性能或更严格的语义**。

### 用法示例

```cpp
#include <ranges>
#include <vector>
#include <iostream>

int main() {
    std::vector<int> vec = {1, 2, 3, 4, 5, 6};

    auto v = vec 
           | std::views::transform([](int x){ return x * 10; })
           | std::views::as_input;          // 降级为 input-only

    for (int x : v) {
        std::cout << x << ' ';   // 10 20 30 40 50 60
    }

    // 以下操作会编译失败（因为是 input-only）：
    // auto it = std::ranges::begin(v) + 2;   // 不支持 random access
    // std::ranges::distance(v);              // 可能不支持（non-common）
}
```

### 关键特性

- **不拥有数据**：像大多数 view 一样，只是对底层 range 的轻量包装。
- **非 common_range**：`begin()` 和 `end()` 返回不同类型（典型 input range 特征）。
- **const 友好**：支持 `const` 迭代（取决于底层）。
- **可组合**：可以和其他 adaptors 链式使用。

### 与类似适配器的对比

| 适配器                | 主要作用                         | C++版本   | 典型场景           |
| --------------------- | -------------------------------- | --------- | ------------------ |
| `views::as_rvalue`    | 强制右值引用                     | C++23     | 移动语义           |
| `views::as_const`     | 强制 const 访问                  | C++23     | 只读视图           |
| **`views::as_input`** | **强制 input-only + non-common** | **C++26** | **性能优化**       |
| `views::common`       | 强制成为 common_range            | C++20     | 需要 common 的算法 |

### Feature Test Macro

```cpp
#if __cpp_lib_ranges_as_input >= 202502L
// 支持 std::ranges::as_input_view / views::as_input
#endif
```

### 总结

`std::views::as_input` 是一个**性能导向**的小工具，用于**故意降低** range 的能力等级，从而让后续算法/适配器选择更轻量、更快的实现路径。它是 C++26 Ranges 持续打磨“可控性能”的一部分。

更多细节可参考：
- cppreference: [`std::ranges::as_input_view`](https://en.cppreference.com/w/cpp/ranges/as_input_view)
- 提案 P3137 / P3828

需要实际性能对比示例或与其他 adaptors 组合使用的场景吗？随时补充！