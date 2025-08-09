# 自定义类型和容器中支持分配器感知

这段代码展示了 C++ 中与内存分配器相关的特性，特别是 C++17 引入的 polymorphic memory resource（PMR，<memory_resource>）框架，以及如何在自定义类型和容器中支持分配器感知（allocator-aware）。代码还涉及异构查找（heterogeneous lookup）和分配器传播的细节。以下是对代码的逐步解释：



```c++
#include <memory_resource>
#include <unordered_map>
#include <string>
#include <vector>
#include <iostream>

// Transparent memory resource that logs allocations
struct TracingResource final : std::pmr::memory_resource {
    TracingResource(std::string_view label, std::pmr::memory_resource* upstream) : label_(label), upstream_(upstream) {}
    std::pmr::memory_resource* upstream() const { return upstream_; }
private:
    // Allocation
    void* do_allocate(std::size_t bytes, std::size_t alignment) override {
        std::cerr << label_ << " allocating " << bytes << " bytes of memory, at " << alignment << " bytes alignment.\n";
        return upstream_->allocate(bytes,alignment);
    }
    // Deallocation
    void do_deallocate(void* p, std::size_t bytes, std::size_t alignment) override {
        std::cerr << label_ << " deallocating " << bytes << " bytes of memory, at " << alignment << " bytes alignment.\n";
        return upstream_->deallocate(p,bytes,alignment);
    }
    bool do_is_equal(const std::pmr::memory_resource& other) const noexcept override {
        auto ptr = dynamic_cast<const TracingResource*>(&other);
        if (ptr == this) return true;
        if (&other == upstream_) return true;
        return false;
    }

    std::string label_;
    std::pmr::memory_resource* upstream_;
};

// Side-note, this custom hash allows us to take a std::string_view
// as argument and use std::unordered_map methods, without
// repeated copies and conversions between std::string,
// std::pmr::string, or even pmr strings that do not use 
// the same backing resource.
// For more context see Heterogenous lookup.
struct StringHash {
  using is_transparent = void;
  [[nodiscard]] size_t operator()(const char *txt) const {
    return std::hash<std::string_view>{}(txt);
  }
  [[nodiscard]] size_t operator()(std::string_view txt) const {
    return std::hash<std::string_view>{}(txt);
  }
  [[nodiscard]] size_t operator()(const std::string &txt) const {
    return std::hash<std::string>{}(txt);
  }
  [[nodiscard]] size_t operator()(const std::pmr::string &txt) const {
    return std::hash<std::pmr::string>{}(txt);
  }
};

// An allocator aware type
struct WordCounter {
    // The demonstration uses std::pmr for brevity, however, 
    // the same approach can be applied to static allocators, 
    // with the caveat that you would need to use
    // std::scoped_allocator_adaptor.
    using allocator_type = std::pmr::polymorphic_allocator<>;

    // Default constructor, with an optional allocator argument
    WordCounter(const allocator_type& alloc = {}) : counter_(alloc) {}
    // Copy constructor
    WordCounter(const WordCounter& other, 
                const allocator_type& alloc = {}) 
      : counter_(other.counter_, alloc) {}
    // Move constructor, note that the move is conditional
    // if (alloc != other.alloc), we have to default to a copy.
    WordCounter(WordCounter&& other, 
                const allocator_type& alloc = {})
      : counter_(std::move(other.counter_), alloc) {}

    // Assignment operators remain without change
    WordCounter& operator=(const WordCounter&) = default;
    WordCounter& operator=(WordCounter&&) = default;

    // For demonstration
    void increment(std::string_view word) {
        if (auto it = counter_.find(word); it != counter_.end())
            ++(it->second);
        else
            counter_.emplace(word, 1);
    }
    void decrement(std::string_view word) {
        if (auto it = counter_.find(word);
            it != counter_.end() && it->second != 0)
            --(it->second);
    }
private:
    // Our storage that gets customized with the allocator
    std::pmr::unordered_map<
        std::pmr::string, uint64_t,
        // custom hash and std::equal_to<>
        // to allow for heterogenous lookup
        StringHash, std::equal_to<>> counter_; 
};

// If we cannot put the allocator as the last argument,
// we can use the std::allocator_arg_t tag.
template <typename... Types>
struct VariadicType {
    using allocator_type = std::pmr::polymorphic_allocator<>;

    // Tag first, followed by the allocator, other arguments follow.
    VariadicType(std::allocator_arg_t, const allocator_type& alloc, 
        Types&&... args) {}
    // And we need to keep the non-allocator version around.
    VariadicType(Types&&... args) {}

    // For copy/move, we can use the normal style
    VariadicType(const VariadicType&, const allocator_type = {}) {}
    VariadicType(VariadicType&&, const allocator_type = {});
};

int main() {
    std::pmr::monotonic_buffer_resource buffer;
    
    // We can wrap allocator aware types in containers and the outer
    // allocator will be correctly applied to the elements.
    // Note: for non-pmr, you would need std::scoped_allocator_adaptor
    TracingResource t1("WordCounter:", &buffer);
    std::pmr::vector<WordCounter> counters(&t1);

    // Construct WordCounter, using the allocator constructor.
    // The vector will allocate memory from the allocator.
    counters.emplace_back();
    // The map will allocate the bucket array and the node using the 
    // allocator. The string fits into small string optimization.
    counters[0].increment("hello");
    // Same as above, except the bucket array is already allocated
    // and the string also needs to allocate.
    counters[0].increment("this string is long enough");

    TracingResource t2("Variadic:", &buffer);
    std::pmr::vector<VariadicType<int,int,int>> variadic(&t2);
    variadic.emplace_back(1,2,3); // Calls the allocator constructor
}
```



------

1. **StringHash 自定义哈希函数**

cpp

```cpp
struct StringHash {
    using is_transparent = void;
    size_t operator()(const char* txt) const { return std::hash<std::string_view>{}(txt); }
    size_t operator()(std::string_view txt) const { return std::hash<std::string_view>{}(txt); }
    size_t operator()(const std::string& txt) const { return std::hash<std::string>{}(txt); }
    size_t operator()(const std::pmr::string& txt) const { return std::hash<std::pmr::string>{}(txt); }
};
```

- **目的**：为 std::unordered_map 提供一个支持异构查找（heterogeneous lookup）的哈希函数。
- **功能**：
  - 支持多种键类型（const char*, std::string_view, std::string, std::pmr::string），避免不必要的字符串拷贝或转换。
  - is_transparent 标记启用异构查找，允许查找时使用与容器键类型不同的类型。
- **优势**：
  - 查找时无需构造 std::pmr::string，直接用 std::string_view 或 const char*。
  - 兼容不同分配器创建的 std::pmr::string。

------

2. **WordCounter 类型（分配器感知）**

cpp

```cpp
struct WordCounter {
    using allocator_type = std::pmr::polymorphic_allocator<>;
```

- **分配器类型**：使用 std::pmr::polymorphic_allocator<>，这是 PMR 框架中的多态分配器，支持动态替换内存资源。

构造函数

cpp

```cpp
WordCounter(const allocator_type& alloc = {}) : counter_(alloc) {}
WordCounter(const WordCounter& other, const allocator_type& alloc = {}) 
    : counter_(other.counter_, alloc) {}
WordCounter(WordCounter&& other, const allocator_type& alloc = {})
    : counter_(std::move(other.counter_), alloc) {}
```

- **默认构造函数**：接受可选的分配器，初始化 counter_。
- **拷贝构造函数**：接受源对象和分配器，复制 counter_ 并使用指定分配器。
- **移动构造函数**：
  - 如果 alloc == other.get_allocator()，直接移动 counter_。
  - 如果分配器不同，默认退化为拷贝（避免跨分配器移动的复杂性）。
- **注意**：PMR 容器在移动时会检查分配器相等性。

赋值运算符

cpp

```cpp
WordCounter& operator=(const WordCounter&) = default;
WordCounter& operator=(WordCounter&&) = default;
```

- 使用默认实现，依赖 counter_ 的赋值行为。

成员函数

cpp

```cpp
void increment(std::string_view word) {
    if (auto it = counter_.find(word); it != counter_.end())
        ++(it->second);
    else
        counter_.emplace(word, 1);
}
```

- 使用 std::string_view 查找和插入，利用 StringHash 的异构查找支持。
- emplace 构造 std::pmr::string 键，使用分配器分配内存。

数据成员

cpp

```cpp
std::pmr::unordered_map<std::pmr::string, uint64_t, StringHash, std::equal_to<>> counter_;
```

- 使用 PMR 版本的 unordered_map，键为 std::pmr::string，支持自定义分配器。
- StringHash 和 std::equal_to<> 启用异构查找。

------

3. **VariadicType（带标记的分配器构造）**

cpp

```cpp
template <typename... Types>
struct VariadicType {
    using allocator_type = std::pmr::polymorphic_allocator<>;
    VariadicType(std::allocator_arg_t, const allocator_type& alloc, Types&&... args) {}
    VariadicType(Types&&... args) {}
    VariadicType(const VariadicType&, const allocator_type = {}) {}
    VariadicType(VariadicType&&, const allocator_type = {});
};
```

- **问题**：当分配器不是最后一个参数时，需要区分普通构造函数和分配器构造函数。
- **解决**：使用 std::allocator_arg_t 标记（一个空结构体）作为第一个参数，表示使用分配器构造。
- **实现**：
  - VariadicType(std::allocator_arg_t, alloc, args...)：分配器构造。
  - VariadicType(args...)：普通构造。
  - 拷贝/移动构造函数保持正常风格。

------

4. **使用 PMR 容器**

cpp

```cpp
std::pmr::monotonic_buffer_resource buffer;
std::pmr::vector<WordCounter> counters(&buffer);
```

- **monotonic_buffer_resource**：
  - 一种只增不减的内存资源，适合临时分配。
  - 不支持释放内存，直到整个资源销毁。
- **std::pmr::vector**：
  - 使用 buffer 作为内存源，分配 WordCounter 对象。

构造和操作

cpp

```cpp
counters.emplace_back();
counters[0].increment("hello");
counters[0].increment("this string is long enough");
```

- **emplace_back()**：
  - 构造 WordCounter，其内部 unordered_map 使用 buffer 分配内存。
- **increment("hello")**：
  - "hello" 适合小字符串优化（SSO），无需额外分配。
  - unordered_map 的桶数组和节点从 buffer 分配。
- **increment("this string is long enough")**：
  - 字符串超长，触发动态分配（仍使用 buffer）。
  - 桶数组已存在，只需分配新节点和字符串内存。

多参数类型

cpp

```cpp
std::pmr::vector<VariadicType<int,int,int>> variadic(&buffer);
variadic.emplace_back(1,2,3);
```

- 调用不带分配器的构造函数，分配器从 variadic 传播到元素。

------

关键点

1. **PMR 框架**：
   - std::pmr::polymorphic_allocator 允许动态替换内存资源（如 monotonic_buffer_resource）。
   - 比传统静态分配器更灵活。
2. **异构查找**：
   - StringHash 避免了键类型的转换，提高效率。
3. **分配器传播**：
   - PMR 容器自动将分配器传递给元素（如 WordCounter 的 unordered_map）。
   - 非 PMR 场景需使用 std::scoped_allocator_adaptor。
4. **性能**：
   - monotonic_buffer_resource 适合连续分配，避免碎片化。
   - SSO 减少小字符串的分配开销。

------

总结

这段代码展示了如何设计分配器感知类型（如 WordCounter），并结合 PMR 容器和自定义哈希函数优化内存使用和查找效率。它还处理了分配器构造的复杂场景（如 VariadicType），为现代 C++ 内存管理提供了实用示例。