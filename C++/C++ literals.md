# C++ literals 

Integer literals in C++ can be surprisingly complex if you care about their type. The type is determined by the combination of the specified suffix (if any), the base (e.g., decimal), and the literal's value.

An alternative to suffixes are macros from the cstdint header that will annotate a literal to map to the corresponding std::(u)int_leastXX_t type.



```c++
#include <utility>
#include <cstdint>

int main() {
    // The lowest ranked type is selected based on the size
    // the actual type is platform specific

    // for decimal base, and no suffix, the type is signed
    auto a1 = 0;
    // decltype(a1) == int

    static_assert(std::is_same_v<decltype(a1), int>);

    auto a2 = 3'000'000'000;
    // decltype(a2) == long

    static_assert(std::is_same_v<decltype(a2), long>);

    // 'u' suffix forces an unsigned type
    auto a3 = 0u;
    // decltype(a3) == unsigned

    static_assert(std::is_same_v<decltype(a3), unsigned>);

    auto a4 = 5'000'000'000u;
    // decltype(a4) == unsigned long

    static_assert(std::is_same_v<decltype(a4), unsigned long>);

    // for non-decimal, unsigned types can be selected 
    // even without the 'u' suffix
    auto a5 = 0x8000'0000; // INT_MAX + 1
    // decltype(a5) == unsigned

    static_assert(std::is_same_v<decltype(a5), unsigned>);

    // Size suffixes

    // Signed integer literals
    auto b1 = 0;
    // decltype(b1) == int

    static_assert(std::is_same_v<decltype(b1), int>);

    auto b2 = 0L; // at least long
    // decltype(b2) == long

    static_assert(std::is_same_v<decltype(b2), long int>);

    auto b3 = 0LL; // C++11, at least long long
    // decltype(b3) == long long

    static_assert(std::is_same_v<decltype(b3), long long int>);

    auto b4 = 0Z; // C++23, guaranteed std::ptrdiff_t
    // decltype(b4) == std::ptrdiff_t

    static_assert(std::is_same_v<decltype(b4), std::ptrdiff_t>);

    // Unsigned integer literals
    auto c1 = 0U; // unsigned type
    // decltype(c1) == unsigned

    static_assert(std::is_same_v<decltype(c1), unsigned int>);

    auto c2 = 0UL; // unsigned type, at least long
    // decltype(c2) == unsigned long

    static_assert(std::is_same_v<decltype(0UL), unsigned long int>);

    auto c3 = 0ULL; // C++11, unsigned type, at least long long
    // decltype(c3) == unsigned long long

    static_assert(std::is_same_v<decltype(0ULL), unsigned long long int>);

    auto c4 = 0UZ; // C++23, guaranteed std::size_t
    // decltype(c4) == std::size_t

    static_assert(std::is_same_v<decltype(c4), std::size_t>);

    // Fixed-sized integer type macro from <cstdint>
    auto d1 = UINT64_C(0); // C++11, guaranteed std::uint_least64_t
    // decltype(d1) == std::uint_least64_t

    static_assert(std::is_same_v<decltype(d1), std::uint_least64_t>);
}
```

