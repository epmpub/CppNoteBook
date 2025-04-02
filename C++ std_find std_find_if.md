# std::find std::find_if

If you want to look up an element in a range by value or by predicate, the most straightforward options are the three linear find algorithms:

- *std::find* (find element by value)
- *std::find_if* (find element using a positive predicate)
- *std::find_if_not* (find element using a negative predicate)

All three algorithms have parallel (C++17) and range variants (C++20).

```c++
#include <algorithm>
#include <vector>
#include <string>
#include <functional>


std::vector<int> data{1, 2, 3, 4, 5, 6, 7};

// Find by value
auto it = std::find(data.begin(), data.end(), 5);
// *it == 5

it = std::find(data.begin(), data.end(), 42);  
// it == data.end()


// Find using a predicate
auto is_even = [](int v) { return v % 2 == 0; };
it = std::find_if(data.begin(), data.end(), is_even);
// *it == 2


// Find using a negation of a predicate
it = std::find_if_not(data.begin(), data.end(), is_even);
// *it == 1

// same as
it = std::find_if(data.begin(), data.end(), std::not_fn(is_even));
// *it == 1


// Range version
it = std::ranges::find(data, 3);
// *it == 3

// With a projection
std::vector<std::string> strs{"a", "bb", "ccc", "dddd"};
// Find the first element that has an even length
auto it2 = std::ranges::find_if(strs, is_even, 
    [](const std::string& str) {
        return str.length();
    });
// *it2 == "bb"
```

[Open the example in Compiler Explorer.](https://compiler-explorer.com/z/1j51xf7fP)