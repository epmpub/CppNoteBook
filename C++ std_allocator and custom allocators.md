# std::allocator and custom allocators

The allocator mechanism in STL: std::allocator and custom allocators.

Allocators are one of the more esoteric parts of C++.

All containers rely on allocators, defaulting to *std::allocator*.

If we want to use a different allocator, we can specify it as the last template argument.

Because template arguments are part of the type, the allocator cannot be easily switched when exposing types through API boundaries.

```C++
#include <boost/pool/pool_alloc.hpp>
#include <list>

void some_func(const std::list<int>&) {}

// A container with an implicit allocator:
std::list<int> list1;
// Equivalent explicit type:
std::list<int, std::allocator<int>> list2;
// decltype(list1) == decltype(list2)

// Customizing the allocator:
using BoostAlloc = boost::fast_pool_allocator<int>;
std::list<int, BoostAlloc> list3;
// Note that boost::pool is not very performant, I would use Intel TBB,
// but that library is currently bugged in Compiler Explorer.

// decltype(list3) != decltype(list1)
// some_func(list3); // Wouldn't compile

// The goal of using a pool allocator is to limit 
// the total number of allocations.
{
std::list<int> list;
for (size_t i = 0; i < 10'000; ++i) list.push_back(i);
}
// 10'000 allocations and deallocations (one for each node)

{
std::list<int, BoostAlloc> list;
for (size_t i = 0; i < 10'000; ++i) list.push_back(i);
}
// ~10 allocations
```

[Open the example in Compiler Explorer.](https://compiler-explorer.com/z/ex1q7z3MK)