#### C++26：std::promise 和 std::packaged_task 的类型擦除分配器（Type-Erased Allocator）行为一致化

（提案 P3503R3）。这是 C++26 中对 <future> 头文件的一个重要清理和一致性改进。1. 历史背景与不一致问题在 C++11 引入 std::promise 和 std::packaged_task 时：

- 两者都支持使用自定义分配器构造共享状态（shared state）。
- 但实现方式和接口存在明显不一致：
  - std::promise 使用类型擦除（type-erased）分配器模型（类似 std::shared_ptr 的 polymorphic_allocator 风格）。
  - std::packaged_task 的 allocator 支持则较为混乱，有些构造函数使用 uses-allocator 构造，有些不支持类型擦除。
  - packaged_task 的 reset() 函数在 allocator 方面的行为与 promise 不匹配。
  - 部分 allocator-aware 构造函数缺失或行为不一致，导致用户代码在两个类之间移植困难。

长期以来，标准库实现和用户都对此感到困惑（LWG Issue 相关讨论较多）。2. C++26 的主要变更（P3503）提案的目标是让两者在类型擦除分配器上的行为完全一致：

- 统一采用类型擦除分配器模型（type-erased allocator）。
- 为 std::promise 和 std::packaged_task 添加/调整 allocator 相关的构造函数。
- 明确 packaged_task::reset() 在使用 allocator 时的行为。
- 移除一些过时或不一致的 uses_allocator 特化（或使其行为一致）。
- 保持向后兼容，同时提供更清晰、统一的接口。

关键新/调整的构造函数示例（简化）：

cpp

```cpp
// std::promise
template<class Allocator>
promise(std::allocator_arg_t, const Allocator& alloc);

// std::packaged_task
template<class R(Args...), class Allocator>
packaged_task(std::allocator_arg_t, const Allocator& alloc, F&& f);
```

两者现在都能可靠地使用 std::pmr::polymorphic_allocator 或其他类型擦除分配器来分配内部共享状态。3. 实际使用示例

cpp

```cpp
#include <future>
#include <memory_resource>

int main() {
    std::pmr::monotonic_buffer_resource mbr;
    std::pmr::polymorphic_allocator<> alloc(&mbr);

    // C++26 前后行为更一致
    std::promise<int> p(std::allocator_arg, alloc);

    auto task = std::packaged_task<int()>(std::allocator_arg, alloc, 
        []{ return 42; });

    // reset() 行为也更一致
    task.reset();                    // 使用原 allocator 重新分配
}
```

4. 主要益处

- 接口一致性：promise 和 packaged_task 在 allocator 使用上行为相同，降低认知负担。
- 更好支持 PMR（Polymorphic Memory Resources）：更容易在内存受限或性能敏感场景中使用自定义内存池。
- 清理历史遗留：移除不一致的实现细节，方便标准库实现者和用户。
- 为未来异步特性铺路：与 C++26 的 std::execution::task、async_scope 等结合更自然。

总结P3503R3 是一个**“一致性修复”提案，虽然功能上不算革命性，但极大提升了 std::promise 和 std::packaged_task 在自定义分配器**场景下的可用性和可预测性。这是 C++26 持续清理和现代化标准库接口的一部分。如果你在写高性能服务器、嵌入式系统或大量使用自定义内存分配器的异步代码，这个变更会让你使用 promise / packaged_task 时更加顺手。需要具体构造函数列表、与 std::pmr 结合的完整示例，还是与之前行为的详细对比吗？