（C++26）std::views::cache_latest 是 Ranges 库中的一个视图适配器（range adaptor），其作用是缓存底层视图中“最近一次解引用（dereference）”得到的元素，避免在同一迭代位置重复计算昂贵的元素值。 



为什么需要它？Ranges 中的许多视图（如 transform、filter 等）是惰性（lazy） 的：每次对迭代器进行 *it 操作时，都会重新执行计算。这在组合视图时会导致重复计算，尤其在 transform 后面接 filter 时特别明显。经典例子（来自提案）：

cpp

```cpp
auto even_squares = v 
    | std::views::transform([](int i) { 
        std::print("transform: {}\n", i); 
        return i * i; 
      })
    | std::views::filter([](int i) { 
        std::print("filter: {}\n", i); 
        return i % 2 == 0; 
      });
```

不使用缓存时，满足条件的元素（如 2 和 4）会被 transform 计算两次（一次在 filter 判断时，一次在循环中获取值时）。使用 cache_latest：

cpp

```cpp
auto even_squares = v 
    | std::views::transform(...) 
    | std::views::cache_latest     // 新增这一层
    | std::views::filter(...);
```

这样 transform 对每个元素只执行一次。 

open-std.org

主要特性

- 模板：template <input_range V> requires view<V> class cache_latest_view;
- 适配器对象：std::views::cache_latest
- 仅支持 input_range（输入范围），不是 forward_range、borrowed_range 或 common_range。
- 缓存机制：
  - 在迭代器的 operator*() 中进行缓存。
  - 每次 ++ 时会重置缓存（因为位置改变了）。
  - 使用 non-propagating-cache 内部实现缓存。
- 引用类型：尽量保留底层 range_reference_t<V> 的引用语义（不是总是拷贝 value_type）。
- 适用场景：底层元素计算昂贵（如复杂函数、I/O、昂贵转换等），且需要重复解引用同一位置时。

使用方式

cpp

```cpp
#include <ranges>

auto cached_view = some_range | std::views::cache_latest;

// 或显式构造
auto v = std::ranges::cache_latest_view(some_range);
```

它支持 begin()、end()、size()（如果底层支持）等常见操作。注意事项

- 线程安全：operator*() 是 const 但会修改内部缓存，因此提案中对数据竞争规则做了有限放宽（input-only 迭代器场景下很少需要跨线程共享）。
- 不适合所有情况：如果元素计算不昂贵，或你只需要前向/随机访问语义，使用它反而可能引入不必要的开销。
- 实现状态：GCC 已开始支持（2025 年左右），其他编译器也会逐步跟进 C++26。 gcc.gnu.org

更多细节可参考：

- [cppreference](https://en.cppreference.com/cpp/ranges/cache_latest_view)
- 提案 [P3138R3](https://www.open-std.org/jtc1/sc22/wg21/docs/papers/2024/p3138r3.html)

这是 C++26 Ranges 改进中一个针对性能痛点的实用特性，尤其适合复杂管道（pipeline）。

```c
#include <ranges>
#include <vector>
#include <print>

int main() {
    std::vector<int> v = {1, 2, 3, 4, 5, 6, 7, 8, 9, 10};
    auto even_squares = v 
    | std::views::transform([](int i) { 
        std::print("transform: {}\n", i); 
        return i * i; 
      })
    |std::views::cache_latest
    | std::views::filter([](int i) { 
        std::print("filter: {}\n", i); 
        return i % 2 == 0; 
      });

    for (int x : even_squares) {
        std::print("result: {}\n", x);
    }

    return 0;
}
```

