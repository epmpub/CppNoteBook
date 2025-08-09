# std::to_address

C++20 utility for obtaining the address of pointers and pointer-like objects: std::to_address.

The *std::to_address* is a simple C++20 utility that provides uniform **access to the address from raw and smart pointers** (and pointer-like objects) without the need to access the underlying value.

Before this utility was introduced, the most uniform way to obtain the address was using *std::addressof(\*p)*, which requires p to point to a valid object.

```c++
#include <memory>
#include <vector>

auto t1 = std::make_unique<int>(1);
int *p1 = std::to_address(t1);
// p1 == t1.get()

std::unique_ptr<int> t2; // empty smart pointer
int *p2 = std::to_address(t2);
// p2 == nullptr

int x = 0, *t3 = &x;
int *p3 = std::to_address(t3);
// t3 == p3

// Also works for contiguous iterators
std::vector<int> rng{1,2,3};
int *p4 = std::to_address(rng.begin());
// p4 == rng.data()
```

[Open the example in Compiler Explorer.](https://compiler-explorer.com/z/araWzzeTo)