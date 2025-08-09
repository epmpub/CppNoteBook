# !!!C++ PMR Polymorphic Memory Resource  library

Daily bit(e) of C++ #295, The C++17 PMR (Polymorphic Memory Resource) library with type erased allocator: std::pmr::polymorphic_allocator.

C++17 introduced the PMR (Polymorphic Memory Resource) library.

The memory resources in the library offer different allocation patterns and can be chained (a resource will use the parent resource to allocate its internal state/buffers).

The type erased *std::pmr::polymorphic_allocator* allows containers using different memory resources to be ABI compatible.

```C++
#include <memory_resource>
#include <list>
#include <string>
#include <array>

// The library offers shorthand aliases in the pmr namespace:
std::pmr::list<int> list1;
// Equivalent full type:
std::list<int, std::pmr::polymorphic_allocator<int>> list2;
// decltype(list1) == decltype(list2)
static_assert(std::is_same_v<decltype(list1),decltype(list2)>);

// The two backend resources:
// std::pmr::new_delete_resource - calls new/delete
// std::pmr::null_memory_resource - throws std::bad_alloc on allocation

// OK, fits into small string optimization:
std::pmr::string s1("hello world",
    std::pmr::null_memory_resource());
// OK, doesn't fit into small string optimization,
// but will call new to allocate.
std::pmr::string s2("this string is long enough",
    std::pmr::new_delete_resource());
try {
    // Will throw, string doesn't fit into small string optimization:
    std::pmr::string s3("this string is long enough",
        std::pmr::null_memory_resource());
} catch (const std::bad_alloc&) {}

// The monotonic_buffer_resource allocates within a buffer.
// Once the buffer is full, another buffer will be allocated
// using the parent resource. Only deallocates on destruction.

// The default instantiation uses the new_delete_resource 
// and no initial buffer.
std::pmr::monotonic_buffer_resource pool1;
std::pmr::list<int> list3(&pool1);

std::array<std::byte, 512*1024> buffer;
// Example of a monotonic buffer using a stack allocated buffer
// as the initial and only memory.
std::pmr::monotonic_buffer_resource pool2(
    buffer.data(), buffer.size(),
    std::pmr::null_memory_resource());
std::pmr::list<int> list4(&pool2);

// The pool_resource manages pools of memory that serve
// for allocations of the same size.
std::pmr::unsynchronized_pool_resource pool3({
    // The pool will allocate blocks*sizeof(chunk) at a time.
    .max_blocks_per_chunk = 64, 
    // Larger requests than this will bypass the pool 
    // and be served directly by the upstream resource.
    .largest_required_pool_block = 512,
});
std::pmr::list<int> list5(&pool3);

// The polymorphic_allocator correctly handles nested containers.
std::pmr::list<std::pmr::string> list6(&pool3);
list6.emplace_back("this string is long enough");
// Both the node and string will be allocated in the pool.

// The library also offers std::pmr::synchronized_pool_resource,
// which is a thread safe version.
```

[Open the example in Compiler Explorer.](https://compiler-explorer.com/z/sd53rheo3)