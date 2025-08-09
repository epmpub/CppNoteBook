# pmr嵌套资源池

​    在 C++ 中，std::pmr（多态内存资源，Polymorphic Memory Resource）支持嵌套的内存资源池。std::pmr 提供了一种灵活的内存管理机制，允许通过 std::pmr::memory_resource 派生类来实现自定义的内存分配策略，而这些内存资源可以嵌套使用。

嵌套的实现方式

1. **嵌套的内存资源**：
   - 你可以创建一个 std::pmr::memory_resource 对象的层次结构。例如，一个内存资源可以依赖另一个内存资源来分配内存。
   - 常见的嵌套方式是将一个内存资源（如 std::pmr::monotonic_buffer_resource）绑定到一个更大的内存池（如另一个 std::pmr::memory_resource 提供的缓冲区）。
2. **典型例子**：
   - std::pmr::monotonic_buffer_resource 是一个常用的内存资源，它可以被配置为从另一个 std::pmr::memory_resource（称为上游资源，upstream resource）获取内存。
   - 你可以创建一个 monotonic_buffer_resource，并将其上游资源设置为另一个 monotonic_buffer_resource 或其他内存资源（如 std::pmr::new_delete_resource）。
3. **代码示例**：

cpp

```cpp
#include <memory_resource>
#include <vector>
#include <array>

int main() {
    // 创建一个底层的内存缓冲区
    std::array<char, 1024> buffer;

    // 使用缓冲区创建一个 monotonic_buffer_resource
    std::pmr::monotonic_buffer_resource pool{buffer.data(), buffer.size()};

    // 创建另一个 monotonic_buffer_resource，依赖于上面的 pool 作为上游资源
    std::pmr::monotonic_buffer_resource nested_pool{&pool};

    // 使用嵌套的内存资源来分配一个 vector
    std::pmr::vector<int> vec{&nested_pool};
    vec.push_back(42);

    return 0;
}
```

嵌套的行为

- **内存分配**：当嵌套的内存资源（如 nested_pool）需要更多内存时，它会从上游资源（如 pool）请求内存。如果上游资源耗尽内存，分配可能会失败（抛出 std::bad_alloc），除非上游资源还能从更上层的资源获取内存。
- **生命周期**：嵌套的内存资源必须在上游资源的生命周期内使用。如果上游资源被销毁，而嵌套资源仍在使用，会导致未定义行为。
- **释放**：std::pmr::monotonic_buffer_resource 不支持单独的释放操作，内存会在资源销毁时统一释放。嵌套资源的内存释放依赖于上游资源的释放策略。

注意事项

1. **性能**：嵌套内存资源可能会增加分配的复杂性，因为每次分配可能需要遍历资源链。选择合适的嵌套深度和资源类型（如 monotonic_buffer_resource 或 pool_resource）对性能至关重要。
2. **线程安全**：std::pmr::memory_resource 本身不保证线程安全。如果多个线程访问嵌套的内存资源，必须自行实现同步机制（如使用 std::pmr::synchronized_pool_resource）。
3. **资源选择**：
   - std::pmr::monotonic_buffer_resource 适合快速分配但不支持单独释放。
   - std::pmr::synchronized_pool_resource 或 std::pmr::unsynchronized_pool_resource 适合需要动态分配和释放的场景。
   - 你可以根据需求选择合适的资源类型来嵌套。

总结

C++ 的 std::pmr 内存资源池可以嵌套，通过将一个内存资源设置为另一个的上游资源来实现。嵌套提供了灵活的内存管理方式，适合复杂的分配场景，但需要注意生命周期、性能和线程安全问题。根据具体需求选择合适的内存资源类型（如 monotonic_buffer_resource 或 pool_resource）来构建嵌套结构。