#  *std::unique* and  std::unique_copy

The *std::unique* algorithm is typically used on a sorted range to produce the list of unique values.

However, the algorithm is more straightforward than that. It can operate on any forward range and will remove consecutive duplicate values (by shifting the elements in the range to the left). Applying *std::unique* to a sorted range results in a range of unique values because all duplicates are consecutive.

The copy variant *std::unique_copy* will emit the unique values through the provided output iterator instead.

Both algorithms provide a C++17 parallel, and C++20 ranges variant.

```
#include <vector>
#include <algorithm>
#include <ranges>
#include <iterator>

std::vector<int> data{1, 2, 2, 3, 2, 3, 3, 1, 1, 1};

// Remove duplicates, by shifting elements forward
// returns an iterator to the new end
auto it = std::unique(data.begin(), data.end());
auto uniq = std::ranges::subrange(data.begin(), it);
// data == {1, 2, 3, 2, 3, 1, ?, ?, ?, ?}
// uniq == {1, 2, 3, 2, 3, 1}

// To actually remove the elements we need to call erase
data.erase(it, data.end());
// data == {1, 2, 3, 2, 3, 1}


data = {1, -2, 2, 3, -2, 3, -3, 1, -1, 1};
std::vector<int> out;

// Copy variant outputs the unique elements through
// the provided iterator
std::unique_copy(data.begin(), data.end(), std::back_inserter(out),
    // Both algorithms support custom comparison function
    [](int l, int r) {
        return std::abs(l) == std::abs(r);
    });
// out == {1, -2, 3, -2, 3, 1}

out.clear();
// Keep in mind that the projection in the C++20 range version 
// only applies to the comparator, and doesn't affect 
// the values written/copied
std::ranges::unique_copy(data, std::back_inserter(out),
    std::equal_to<>{},
    [](int l) { return std::abs(l); });
// out == {1, -2, 3, -2, 3, 1}
```

[Open the example in Compiler Explorer.](https://compiler-explorer.com/z/hxa9MbxnY)