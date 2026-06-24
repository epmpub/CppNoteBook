**C++26 `std::ranges::approximately_sized_range` 和 `std::ranges::reserve_hint`**

 是 Ranges 库中针对**懒惰范围（lazy ranges）**性能优化的重要补充，通过提案 **P2846R2**（*Eagerly reserving memory for not-quite-sized lazy ranges*）引入。

### 1. `std::ranges::approximately_sized_range`

这是一个**范围概念（range concept）**，C++26 新增。

#### 定义
一个范围 `R` 满足 `approximately_sized_range` 当且仅当它能**在常数时间（amortized constant time）内提供一个合理的元素数量估计**。

```cpp
template<class R>
concept approximately_sized_range = 
    ranges::range<R> && 
    requires(R& r) {
        { ranges::reserve_hint(r) } -> std::integral;
    };
```

- 它**比 `sized_range` 更弱**：`sized_range` 要求 `ranges::size(r)` 返回**精确**大小；而 `approximately_sized_range` 只要求返回一个**估计值**（hint），用于优化内存分配。
- 许多“懒惰”或“生成式”视图（如某些 `transform_view`、`filter_view` 的组合、生成器等）无法精确知道大小，但可以给出合理估计。

### 2. `std::ranges::reserve_hint`

这是一个**定制点对象（customization point object）**，C++26 新增。

#### 作用
返回一个适合用于 `container.reserve(...)` 的**大小提示（hint）**。

```cpp
constexpr auto reserve_hint(const range auto& r);   // C++26
```

**行为规则**（优先级从高到低）：

1. 如果 `ranges::size(r)` 合法 → 返回 `ranges::size(r)`（精确大小）。
2. 如果范围满足 `approximately_sized_range` → 返回 `ranges::reserve_hint` 的自定义实现（通常是估计值）。
3. 否则，返回 `0`（或实现定义的合理默认值）。

### 主要使用场景

```cpp
#include <ranges>
#include <vector>

auto lazy_view = /* some complex lazy pipeline, e.g. filter + transform */;

std::vector<T> result;
result.reserve(ranges::reserve_hint(lazy_view));  // C++26 优化

ranges::copy(lazy_view, std::back_inserter(result));
```

**关键受益**：
- `std::ranges::to<Container>()`（C++23+）现在会自动使用 `reserve_hint`（C++26）。
- 许多视图适配器（如 `chunk_view`、`stride_view` 等）可以提供合理的 hint，避免多次 realloc。

### 示例

```cpp
#include <ranges>
#include <vector>
#include <iostream>

int main() {
    auto nums = std::views::iota(1, 1000)
              | std::views::filter([](int x){ return x % 3 == 0; })
              | std::views::transform([](int x){ return x * 2; });

    std::vector<int> v;
    v.reserve(std::ranges::reserve_hint(nums));   // 使用估计大小预分配

    for (int x : nums) v.push_back(x);

    std::cout << "Reserved: " << v.capacity() << ", size: " << v.size() << '\n';
}
```

### 设计动机

- 很多 lazy ranges **无法**精确知道 `size()`（例如无限生成器、过滤器等），但可以轻松估计一个上限或近似值。
- 提前 `reserve()` 可以显著减少内存重新分配，提高性能。
- 这是对 `sized_range` 的实用扩展，填补了“知道大概大小但不确定精确大小”的空白。

### Feature Test Macro

```cpp
#if __cpp_lib_ranges_reserve_hint >= 202502L
// 支持 approximately_sized_range 和 reserve_hint
#endif
```

**参考**：
- cppreference: [`std::ranges::approximately_sized_range`](https://en.cppreference.com/w/cpp/ranges/approximately_sized_range) 和 [`std::ranges::reserve_hint`](https://en.cppreference.com/w/cpp/ranges/reserve_hint)
- 提案 P2846R2

这是一个非常实用的**性能优化**特性，尤其适合复杂 ranges pipeline 和 `ranges::to<>` 转换场景。需要具体视图如何实现 `reserve_hint` 的示例吗？