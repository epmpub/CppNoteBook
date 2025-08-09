# constexpr keyword

C++17 *constexpr if* replaces most cases where macros would have to be used.

It also simplifies generic functions and metaprogramming and works with C++20 concepts to allow code to adjust based on features supported by the type of the function arguments.

```C++
#include <ranges>

constexpr inline bool feature_x24_enabled = true;

// Macro replacement
void some_function() {
    if constexpr (feature_x24_enabled) {
        // something special for feature x24
    }
}

// Function returning different types based on type of argument
auto generic_function(auto x) {
    if constexpr (std::is_integral_v<decltype(x)>) {
        // argument integral -> returns std::string
        return std::to_string(x);
    } else if constexpr (std::is_floating_point_v<decltype(x)>) {
        // argument floting point type -> returns int64_t
        return static_cast<int64_t>(x);
    } else {
        // otherwise return void
        return;
    }
}

// Works with C++20 concepts:
void custom_algorithm(auto&& rng) {
    using type = decltype(rng);
    if constexpr (std::ranges::random_access_range<type>) {
        // implementation for random_access_range
    } else if constexpr (std::ranges::bidirectional_range<type>) {
        // implementation for bidirectional_range
    } else {
        // fallback implementation
    }
}
```

[Open the example in Compiler Explorer.](https://compiler-explorer.com/z/vT99WMoTr)