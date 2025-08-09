#  std::ranges::to

The C++23 *std::ranges::to* is a simple utility that eagerly evaluates a view and stores the result in the specified container type.

The element type can be left out and will then be deduced using CTAD (matching the element type of the view).

```C++
#include <ranges>
#include <list>
#include <vector>
#include <spanstream>

std::list<int> data{1,2,3,4,5};
auto v = data | std::ranges::to<std::vector>();
// v == {1,2,3,4,5}
// decltype(v) == std::vector<int>


// Composed views
std::span<const char> text = "42 -13 7 3 -1 91 101";
std::ispanstream s(text);
auto parsed = std::views::istream<int>(s) | 
  std::views::filter([](int e) {
    return e >= 0;
  }) | std::ranges::to<std::vector>();
// parsed == {42, 7, 3, 91, 101}
// decltype(parsed) == std::vector<int>


std::string str = "hello world";
auto upper = str | std::views::transform([](char c) {
    return std::toupper(c);
}) | std::ranges::to<std::string>();
// upper == "HELLO WORLD"


// With an explicit element type
std::vector<double> floats{2.4,1.1,9.7,6.3};
auto ints = floats | std::ranges::to<std::vector<int>>();
// ints == {2, 1, 9, 6}
```

[Open the example in Compiler Explorer.](https://compiler-explorer.com/z/53eYs19G4)