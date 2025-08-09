# !!! Implementing custom allocators

Allocators allow us to control the memory allocation patterns of standard containers.

Preferably, one should use well-established allocators. However, implementing a custom allocator from scratch is not complicated.

Keep in mind that stateful allocators increase the size of each object that uses this allocator. The Monostate pattern can potentially prevent this overhead.

```c++
#include <list>
#include <string>
#include <array>
#include <scoped_allocator>

// Backend storage for our allocator
struct StackBuffer {
    // 512kB buffer
    alignas(alignof(std::max_align_t)) 
      std::array<std::byte,512*1024> buffer;
    size_t used = 0;

    // Calculate the required offset for allocating type T
    // so that T is properly aligned.
    template <typename T> size_t get_offset() const {
        if (used % alignof(T) == 0) return 0;
        return alignof(T) - (used % alignof(T));
    }
    // Allocate sizeof(T)*cnt bytes in the buffer, properly aligned.
    template <typename T> T* allocate(std::size_t cnt) {
        size_t off = get_offset<T>();
        if (used + off + cnt*sizeof(T) >= buffer.size())
            throw std::bad_alloc(); // The buffer is full.

        // Pointer to the start of the allocated block.
        T* result = reinterpret_cast<T*>(buffer.data()+used+off);
        used += off + cnt*sizeof(T);
        return result;
    }
    // Deallocation is a no-op.
    void deallocate(void*, std::size_t) {}
};

// Our custom allocator
template <typename T> struct StackAllocator {
    using value_type = T; // Required

    // We need to initialize the first allocator with
    // our buffer.
    StackAllocator(StackBuffer* storage) : storage_(storage) {}
    StackAllocator(const StackAllocator&) = default;
    // Conversion constructor that passes the buffer along.
    template <typename U>
    StackAllocator(const StackAllocator<U>& other) :
    storage_(other.storage_) {}

    // Required
    T* allocate(std::size_t n) {
        return storage_->allocate<T>(n);
    }
    // Required
    void deallocate(T* p, std::size_t n) {
        storage_->deallocate(p,n);
    }
private:
    // Required for the conversion constructor.
    template <typename U> friend struct StackAllocator;
    // Pointer to the buffer, note that this will increase
    // the size of each container that uses this allocator 
    // by sizeof(StackBuffer*), e.g. 8 bytes on x86-64.
    StackBuffer* storage_;
};


// String that doesn't fit into small string optimization.
std::string_view long_string = "This is a long enough string.";

// Create an instance of our buffer, allocating 512kB on the stack.
StackBuffer buffer;

// Allocator that allocates char.
using CharAllocator = StackAllocator<char>;
// String type that uses use the above allocator.
using String = std::basic_string<char, std::char_traits<char>,
         CharAllocator>;

// Instead of allocating on heap, grabs 30 bytes from our buffer.
String string_that_allocates(long_string, CharAllocator(&buffer));
// With class template decuction, this gets more reasonable.
std::basic_string string_ctad(long_string, CharAllocator(&buffer));
// decltype(string_that_allocates) == decltype(string_ctad)

// Allocator that allocates the above String.
using StringAllocator = StackAllocator<String>;

// When we use nested containers, we would normally
// have to manually pass in the allocator.
std::list<String, StringAllocator> manual{StringAllocator(&buffer)};
// Two allocations:
// 30 bytes for the string
// 56 bytes for the node (sizeof(std::string)+Allocator+Next+Prev)
manual.emplace_back(long_string, CharAllocator(&buffer));
// Note that despite passing in an allocator for String, the list
// actually uses a converted allocator that allocates nodes.

// However, the std::scoped_allocator_adaptor can do this for us:
using ThroughAllocator = 
  std::scoped_allocator_adaptor<StackAllocator<String>>;

std::list<String, ThroughAllocator>
 automatic{ThroughAllocator(&buffer)};
automatic.emplace_back(long_string);
// Note that the emplace is important here as it creates the object
// in place, a push_back would create the string on the call site.

// Wouldn't compile:
// automatic.push_back(long_string);
```

[Open the example in Compiler Explorer.](https://compiler-explorer.com/z/dcvT6WPrP)