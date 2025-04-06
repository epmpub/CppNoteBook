# *std::as_const* 

*std::as_const* is a C++17 utility that simplifies const-casting, 

specifically the safe variant of adding a const qualifier.

Notably, this utility is more ergonomic in generic contexts than the standard *const_cast*.

```c++
#include <utility>

void test(const auto&) {}
void test(auto&)  {}

int x = 0;

// If we want to explicitly call test(const&)
// we have to add const qualifier:
test(const_cast<const int&>(x)); // old style
test(std::as_const(x)); // shorter and can't mispell the type

void user(auto&& x) {
    // In generic code, ensuring that we call test(const&)
    // is quite tricky to get right.
    test(const_cast<const std::remove_reference_t<decltype(x)>&>(x));
    // as_const provides a lot shorter and correct solution
    test(std::as_const(x));
}

user(x); // Calls test(const&) twice
user(10); // Same as above
```

[Open the example in Compiler Explorer.](https://compiler-explorer.com/z/sP5MeEP6b)