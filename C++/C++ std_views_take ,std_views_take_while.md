#  *std::views::take* ,*std::views::take_while*

The C++20 *std::views::take* and *std::views::take_while* produce views of the leading elements of the adapted range.

Additionally, *std::views::take* is optimized for random access ranges and will not introduce any overhead for *std::span*, *std::string_view*, *std::views::iota*, *std::views::repeat* and *std::ranges::subrange* (if it models *random_access* and *sized*).

```c++
#include <ranges>
#include <vector>


std::vector<int> data{1,2,3,4,5}; 

// First three elements of data
auto x = data | std::views::take(3);
// x == {1,2,3}
// decltype(x) == std::ranges::take_view<std::vector<int>>


// Taking more elements than available is safe
auto y = data | std::views::take(42);
// y = {1,2,3,4,5}
// std::size(y) == 5


// Optimized for span, string_view, views::iota, 
// views::repeated, ranges::subrange.
auto z = std::span(data) | std::views::take(3);
// z == {1,2,3}
// decltype(z) == std::span<int>


// Take until the predicate evaluates as false
auto v = data | std::views::take_while([](int i) {
    return i % 5 != 0;
});
// v == {1,2,3,4}
```

[Open the example in Compiler Explorer.](https://compiler-explorer.com/z/Efv174Yaf)